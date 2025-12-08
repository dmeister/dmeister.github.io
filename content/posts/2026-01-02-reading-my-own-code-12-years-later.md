---
title: "Reading My Own Code 12 Years Later: The Append-Optimized Story"
date: 2026-01-02
mermaid: true
---

*A journey of rediscovering my first major contribution to production databases through code archaeology*

## I Remember Building This (But Not How)

Summer 2013. I had just finished my PhD in storage systems. I rejoined Greenplum (I'd interned there in 2012) to work on their storage and transactions team. Between the internship and finishing the PhD, Greenplum changed. Hadoop was everywhere. The company was in turmoil.

My task: extend Greenplum's append-only tables to support UPDATE and DELETE—not for high-frequency updates, but to make them *possible* when needed. Keep the performance benefits for the primary workload: bulk loads and fast scans.

I was the sole engineer on the project. It became a headline feature of [Greenplum Database 4.3](https://docs.huihoo.com/greenplum/pivotal/4.3.6/relnotes/GPDB_4300_README.html#topic5). By that time, I had already joined Pure Storage.

Fast forward to 2026. Greenplum is now open source (or was). The code I wrote 13 years ago is on GitHub at [github.com/greenplum-db/gpdb-archive](https://github.com/greenplum-db/gpdb-archive). I remember the big picture: visibility maps, compaction, segment files—but the details? Those faded long ago.

So I did something both strange and enlightening: I treated my own code like an archaeological dig (with the help of git history and Claude Code). Reading through it, memories surfaced—some clear, some fuzzy. But I also discovered things that changed after I left, evolution I never witnessed, and decisions that only make sense now with context I didn't have then.

*Note: All technical details, code snippets, and git history references in this post come from the open source Greenplum codebase at [github.com/greenplum-db/gpdb-archive](https://github.com/greenplum-db/gpdb-archive).*

This is the story of what I found, what I remembered, and what surprised me.

## The 2013 Context: Hadoop, Layoffs, and LSM-Trees

To understand the design, you need to understand the moment.

Greenplum Database was a massively parallel processing (MPP) analytical database built on PostgreSQL—a data warehouse for petabyte-scale analytics. It split data across hundreds of segment nodes, each a modified PostgreSQL instance. Unlike Hadoop's batch MapReduce, Greenplum offered full SQL, ACID transactions, columnar storage, and 5-10x compression. It competed with Teradata, Vertica, and Netezza. EMC acquired it in 2010 for $400M.

But 2013 was turbulent. Hadoop had changed everything. The conventional wisdom said SQL databases were dead, NoSQL and MapReduce was the future. Layoffs followed. The storage team I'd worked with during my 2012 internship was decimated. When I came back, only three or so remained.

Greenplum already had append-only tables optimized for bulk loads: write-once segment files, columnar storage, 5-10x compression, fast sequential scans. The key was immutability—data was written sequentially to segment files and **never modified**. This enabled aggressive compression and lightning-fast sequential scans. Perfect for bulk INSERT operations.

But no UPDATE or DELETE. Once written, data was permanent. For data warehouses, this created real problems. Late-arriving facts needed updates. Data quality corrections required deletes. Slowly-changing dimensions needed both. The workarounds were painful: drop and reload entire partitions (potentially TB of data to fix a few rows), or complex application-level change tracking.

These weren't going to be frequent operations—data warehouses are mostly read and append. But they needed to *work* when needed.

The challenge was clear: make UPDATE and DELETE possible while keeping the append-only performance benefits. Not optimized for update-heavy workloads—just make them work when needed. Essentially, build a log-structured merge table for a parallel analytical database where updates are relatively rare.

I knew the log-structured concept from working with SSDs during my PhD. Flash storage loves sequential writes, hates random updates. The log-structured file system paper ([Rosenblum and Ousterhout, 1991](https://web.stanford.edu/~ouster/cgi-bin/papers/lfs.pdf)) had codified the core idea: sequential writes are fast, use a log, compact later. LSM-trees ([O'Neil et al., 1996](https://paperhub.s3.amazonaws.com/18e91eb4db2114a06ea614f0384f2784.pdf)) extended this for databases.

But applying it here was different. This wasn't a single-node system but a distributed one. It was analytical, not transactional. It was PostgreSQL-based, so MVCC semantics were required. And it needed to build on existing append-only infrastructure.

PostgreSQL's heap tables use MVCC with `xmin` and `xmax` fields in every tuple—16 bytes of overhead per row. These are transaction IDs that track visibility: xmin records which transaction created the row, `xmax` records which transaction deleted or updated it. Every query uses these fields to determine whether a row is visible to its snapshot. Append-only tables avoided this overhead by not supporting updates at all—no xmin and xmax fields needed when rows are write-once.

We needed something in between: MVCC semantics without per-tuple overhead. An LSM-tree that played nice with PostgreSQL's transaction system.

## The Design: LSM-Tree Meets PostgreSQL MVCC

My PhD wasn't about LSM-trees—it was on data deduplication for backup systems. But working on storage systems for SSDs, you absorb these ideas. Log-structured designs were everywhere in the SSD world.

Reading the code now, I remember the central insight: physical storage stays append-only (log-structured), but visibility becomes a metadata problem (MVCC on the metadata, not the data). Log-structured writes mean appending to segment files, never updating in place. Merge and compaction periodically reclaim space. A metadata overlay uses PostgreSQL's MVCC on visibility metadata instead of tuple headers.

The existing append-only tables already had the "log" part—sequential writes to segment files. What they lacked was a way to track deletes and a compaction strategy that worked with distributed transactions.

### The Architecture

The central insight: physical storage stays append-only (log-structured), but visibility becomes a metadata problem. Instead of marking tuples as deleted in-place (PostgreSQL's approach), track visibility separately using MVCC on the metadata, not the data.

{{< mermaid >}}
flowchart TD
    subgraph SegmentFiles["Segment Files (Append-Only)"]
        SF0["SF 0<br/>All Rows"]
        SF1["SF 1<br/>All Rows"]
        SFn["SF n<br/>All Rows"]
    end

    subgraph VisibilityMap["Visibility Map (Heap Table)"]
        Bitmap["Bitmap: Which rows visible?<br/>Uses PostgreSQL MVCC"]
    end

    SegmentFiles -->|Physical data<br/>never changes| Note1[" "]
    Note1 -.->|" "| VisibilityMap
    VisibilityMap -->|Metadata controls<br/>what you see| Result[" "]

    style SegmentFiles fill:#e1f5ff,stroke:#333,stroke-width:2px
    style VisibilityMap fill:#fff5e1,stroke:#333,stroke-width:2px
    style Note1 fill:none,stroke:none
    style Result fill:none,stroke:none
{{< /mermaid >}}

When you **DELETE** a row, nothing happens to the segment file physically—the metadata simply marks the row invisible in the visimap. When you **UPDATE** a row, the physical layer appends a new version to the segment file while the metadata layer marks the old version invisible. The result: MVCC semantics with just 1 bit of overhead per row instead of 16 bytes.

The design had three key components:

**1. Segment Files (leveraged existing infrastructure):**
- Physical storage: up to 128 segment files per table (0-127)
- Each containing variable-sized blocks with optional compression
- **Once written, never modified** (log-structured)
- Each row gets a unique identifier: `(segment_file_number, row_number)`

**2. Visibility Map (the central idea):**

Instead of storing visibility in tuple headers like PostgreSQL's heap tables, I created a separate heap table (`pg_aovisimap_<oid>`) where:
- Each entry covers **32,768 rows** (`2^15`)
- Contains a compressed bitmap: **1 bit per row** (`1` = visible, `0` = deleted)
- **Uses PostgreSQL's MVCC for free** by storing the bitmap in a heap table

The overhead comparison is dramatic:
- **PostgreSQL heap tables:** 16 bytes per row (`xmin` + `xmax`) → 16 GB per billion rows
- **Append-only visimap:** ~1 bit per row → ~1 MB per billion rows

**3. Metadata Tables (for coordination):**
- `pg_aoseg_<oid>`: Segment file metadata (EOF, tuple counts, state)
- `pg_aoblkdir_<oid>`: Block directory for index support
- All heap tables, all get MVCC automatically

The elegance was in leveraging PostgreSQL's existing MVCC for metadata instead of reinventing it for the data. By making the visimap itself a versioned heap table, different transactions automatically see different versions of the visibility bitmap—MVCC semantics with minimal implementation complexity.

## How Visibility Works: The Visimap in Action

I remember the concept: use a bitmap to track visibility. But reading through `appendonly_visimap_entry.c`, the details come flooding back—details I'd completely forgotten:

```c
typedef struct AppendOnlyVisimapEntry {
    int32  segmentFileNum;    // Which segment file (0-127)
    int64  firstRowNum;       // First row this entry covers
    bytea  bitmap;            // Compressed bitmap (32,768 bits max)
    // Stored in pg_aovisimap_<oid> heap table
} AppendOnlyVisimapEntry;
```

Consider `DELETE FROM fact_table WHERE order_date = '2013-06-15'`. For a row at `segfile 2`, `rownum 100000`: calculate which visimap entry covers it (firstRowNum = 98304), load that entry, set bit 1696 to 0 (invisible), update the visimap tuple via MVCC.

This works across transactions because the visimap itself is versioned. Transaction T1 sees visimap version V1 (row visible). Transaction T2 deletes, creating version V2 (row invisible). Transaction T3 sees V2 (row invisible). Different snapshots see different visimap versions—MVCC for free.

Reading this code now, I see `APPENDONLY_VISIMAP_MAX_RANGE = 32768`. I have no recollection how I arrived at this number. It's `2^15`, so the division is a simple bit shift (`rownum >> 15`), and it's large enough to amortize overhead but small enough to avoid wasting space on sparse deletes. But did I analyze different values? Run experiments? Just pick a round number? I don't remember. The code doesn't explain, and my memory is silent.

## How Compaction Works: The Merge in LSM-Tree

DELETEs mark rows invisible, but don't reclaim disk space. That's where VACUUM and compaction come in—the "merge" part of log-structured merge trees. This wasn't my PhD research area, but the concepts were familiar from working with SSDs. I remember this being the hardest part to get right.

The core idea: for each segment file, calculate the hide ratio (`invisible_rows` / `total_rows`). If it exceeds a threshold (default 10%), scan through all rows, check the visimap for visibility, write visible rows to a new segment and skip invisible ones, mark the source segment `AWAITING_DROP`, and update metadata. The implementation is iterative within a single transaction:

```c
insert_segno = -1;
while ((compaction_segno = ChooseSegnoForCompaction(onerel,
                                   compacted_and_inserted_segments)) != -1)
{
    compacted_segments = lappend_int(compacted_segments, compaction_segno);

    if (RelationIsAoRows(onerel))
        AppendOnlyCompact(onerel, compaction_segno, &insert_segno,
                         isFull, compacted_segments, vacrelstats);

    if (insert_segno != -1)
        compacted_and_inserted_segments = list_append_unique_int(
            compacted_and_inserted_segments, insert_segno);

    CommandCounterIncrement();  // See our own changes
}
```

All segments are compacted in one transaction for consistency, but this requires up to 2x disk space temporarily. I left this comment in the code:

```c
/*
 * XXX: Because we compact all segfiles in one transaction, this can
 * require up to 2x the disk space. Alternatively, we could split this
 * into multiple transactions. The problem with that is that the updates
 * to pg_aoseg needs to happen in a distributed transaction...
 */
```

Reading this now, I don't recall extensively analyzing the space overhead. A leveled compaction approach (like typical LSM-trees use) would consume less space—compact one level at a time instead of all segments at once. But it would add significant complexity: more state to track, more transaction coordination, more edge cases. I appear to have chosen simplicity: compact everything in one transaction, accept the 2x disk space requirement.

## The PostgreSQL Architecture Challenge

One thing reading the code doesn't fully capture: how difficult PostgreSQL's architecture made background operations. This wasn't LSM-tree theory—this was wrestling with a database architecture designed in the 1990s.

PostgreSQL uses processes, not threads. Every connection gets its own OS process. No shared state without explicit shared memory, no background threads, compaction had to happen in VACUUM, process coordination required IPC. The thread-per-connection model (MySQL, SQL Server) was already common in 2013, but PostgreSQL's process model was architectural bedrock.

PostgreSQL allocates all shared memory at startup—no dynamic allocation. Every data structure visible across processes needed pre-allocated shared memory. I had to estimate worst-case memory usage at design time, with constant trade-offs between wasting memory and running out during compaction.

The entire codebase was C—no C++, no modern conveniences. Manual memory management (palloc and pfree everywhere). No RAII, no smart pointers. Error handling via PostgreSQL's `longjmp`-based `ereport(ERROR)`. Pointer arithmetic and casting everywhere. Actually, I introduced at least one bad bug due to wrong casting from int64 to int32. I saw this bug while reading the code for this post. The bug was found and fixed in 2018.

PostgreSQL (and Greenplum) relied heavily on globals: `MyProc`, `MyDatabaseId`, `CurrentMemoryContext`, `ActiveSnapshot`, and dozens more. For a new engineer, this was disorienting. You'd read a function and wonder where values came from. The answer: a global variable 5 files away, set during process initialization. Modern languages encourage dependency injection. PostgreSQL's 1990s C codebase used globals liberally.

The combination made compaction tricky: run a multi-phase operation (scan, write, update metadata) across a distributed database, coordinating processes with pre-defined shared memory, within a user-initiated `VACUUM`. The solution leveraged PostgreSQL's existing infrastructure: VACUUM's distributed transaction framework, `CommandCounterIncrement()` to see our own changes mid-transaction, the catalog system for coordination, and accept the 2x disk space requirement for simplicity. It wasn't elegant, but it worked with the architecture, not against it.

## Standing on Shoulders: What I Built On

Reading the code now, I can see how much I leveraged existing infrastructure. This philosophy—don't rebuild what already works—is probably why I could ship it in a few months as the sole engineer.

From existing append-only tables, I inherited: segment file format with variable-sized blocks, compression infrastructure (zlib, RLE), columnar storage, AOTupleId encoding (segfile, rownum) in 48 bits, and the entire write path with checksums and compression.

What I added: the visibility map system with bitmap encoding and MVCC integration, the compaction algorithm, index support and support tooling.

The consistent philosophy: don't reinvent what PostgreSQL does well. Use heap tables for metadata (MVCC for free), PostgreSQL's WAL (crash recovery), 2PC (distributed transactions), and snapshot infrastructure.

This made implementation simpler and more reliable. But I also inherited PostgreSQL's limitations.

## 12 Years Later: What Still Works

Diving into the current codebase, I was surprised by how much survived. The fundamental LSM-tree design is still there in 2026: segment files still append-only, visimap still using 32,768-row entries, three-phase VACUUM unchanged, metadata tables still using heap storage. The abstractions held up. Or at least, were never replaced.

The benefits are still real: 5-10x compression, fast sequential scans, efficient bulk loads, low memory overhead (1MB per billion rows). The separation of concerns—physical storage versus visibility, data versus metadata—held up well.

These abstractions enabled later features without fundamental redesigns: column-oriented storage enhancements, unique indexes on AO tables (2022), fast ALTER TABLE ADD COLUMN (2023), index-only scans (2023). Reading this evolution—features I never knew existed—validates the abstractions. The team could extend without rewriting.

I left bugs. Let's be honest. I likely left much more, but here's one I found in the git history. In appendonly_visimap_entry.c, I wrote:

```c
int64 AppendOnlyVisimapEntry_GetFirstRowNum(
    AppendOnlyVisimapEntry *visiMapEntry,
    AOTupleId *tupleId)
{
    int rowNum;  // Should be int64!

    rowNum = AOTupleIdGet_rowNum(tupleId);
    return (rowNum / APPENDONLY_VISIMAP_MAX_RANGE) * APPENDONLY_VISIMAP_MAX_RANGE;
}
```

Classic type mismatch: the function returns `int64`, but I used `int` for the local variable. C compiles without warnings (implicit conversions are silent). Works for 2 billion rows and then it breaks. Manual testing missed it, one variable among thousands of lines. I remember being careful about types, but not careful enough.

Modern development would catch this: `clang --analyze -Wconversion` would warn, but the DevEx tooling was limited.

## Where Greenplum Is Now

The GitHub repository is called gpdb-archive for a reason. Open source development stopped around 2024. Greenplum still exists as [VMware Tanzu Greenplum](https://www.vmware.com/products/app-platform/tanzu-greenplum.html). Some engineers I worked with are still there. I even saw a [LinkedIn post](https://www.linkedin.com/posts/ivannovick_vmware-tanzu-greenplum-is-growing-and-hiring-activity-7406740863931793408-rCD0?utm_source=share&utm_medium=member_desktop&rcm=ACoAAAPLi_UBnX3TGNIuVmn3Jz98tjmOi3M8p04) recently—they're hiring.

But in the grand scheme, Greenplum didn't succeed the way competitors did. Consider Redshift: similar background (analytical database, columnar storage, MPP architecture), but it transitioned to cloud-native successfully. Greenplum struggled. I think that is fair to say in hindsight. I am happy that it still exists and people still use it, but it didn't succeed as well as others.

Reading this old codebase, I think, even at the time, dated engineering practices were part of the problem. Not the only problem—market dynamics, company changes, strategic decisions all mattered. But the technical foundation matters more than people think. When the world shifted to cloud-native, auto-scaling, serverless, a codebase built on 1990s PostgreSQL with process-per-connection, pre-defined shared memory, no unit tests, and mixed coding styles made adaptation harder.

This taught me something I didn't fully appreciate in 2013: even in crisis mode, even when shipping quickly matters most, you can only take on so much technical debt before it becomes an anchor. No startup needs perfect engineering. But some baseline level of testability, tooling, and discipline around complexity—these aren't luxuries. They determine whether you can adapt when the world changes. And the world will change.

When I switched to Pure Storage that year, I found a modern C++ codebase with a high-performance custom async framework and at least reasonable testing, which was much easier to work with.

I look at modern data warehouses—Snowflake, BigQuery, newer versions of Redshift—and they're not just better at cloud economics. They're built on foundations that make iteration easier. Better languages (Go, Rust, C++ instead of pure C), better testing practices, better separation of concerns, better tooling. The technical decisions compound over decades.

Would better engineering practices have saved Greenplum? Maybe not. Market dynamics are powerful. But they would have made the fight easier. In the end, Hadoop didn't win long term. SQL databases eventually came back, reborn in the cloud. I will literaly start next week working on a SQL database again.

## Code as Time Capsule

Reading my own code 13 years later is a strange experience. It's simultaneously familiar and foreign. I remember the architecture—the LSM-tree design, the visibility map idea, the three-phase compaction. But the details? The specific functions, the edge cases, the bug fixes, the comments explaining subtle race conditions? Those are gone.

How much of engineering is forgetting. We build these complex systems, understand them deeply for a moment, then move on. The code remains, but the mental model fades. The details blur. Eventually, reading our own code feels like archaeology. This is why open source is remarkable. When Greenplum was closed-source, this code was locked away. I couldn't have done this exercise. But once it was open-sourced, it became archaeological evidence. Not just for others, but for me—recovering my own past through git commits and code comments.

## What I Learned from This Exercise

The LSM-tree based design for append-optimized tables held up for 13 years because the core abstractions—log-structured segment files, visibility metadata, merge and compaction—were right for the problem. I just recognized that the same principles applied here as they did for SSD storage. Often engineering is less about invention and more about recognizing which existing ideas fit your constraints.

By using PostgreSQL's MVCC for metadata instead of building my own transaction system, I saved months of work and inherited decades of battle-testing. Often the best code is the code you don't write. But I also inherited PostgreSQL's limitations: 2PC coordination, shared memory constraints, process model complexity. These would cause issues later.

The lack of unit tests meant slow feedback loops, difficulty testing edge cases, no safety net for refactoring. The int64 overflow bug survived 6 years partly because we couldn't easily test billion-row scenarios. Modern CI/CD with fast unit tests and property-based testing would have caught it immediately. Even in 2013, I knew this was suboptimal. But you work with what you have. Invest in testing infrastructure early. Your future self will thank you.

The comments I left were breadcrumbs to my reasoning. They explained why, not just what. That comment about compacting all segfiles requiring 2x disk space? Documents a significant tradeoff I barely remember making. Without it, I'd wonder: "Why not leveled compaction?" The comment preserves the reasoning: distributed transaction complexity wasn't worth the space savings. Was that really true? No idea, but at least that I what I thought at the time.

Code persists longer than memory. The append-optimized feature is still running in production systems worldwide, maintained by people who never met me.

## Acknowledgments

The Greenplum storage team in 2013 trusted a fresh PhD to build a critical feature. The maintainers kept this code running and improving for over a decade. The community made the code open source, allowing this archaeological expedition.
