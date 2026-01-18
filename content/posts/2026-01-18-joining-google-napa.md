---
title: "Joining Google: From 2008 Rejection to the Napa Team"
date: 2026-01-18
---

On January 5, I started at Google as a Software Engineer on the Napa team.

The journey here has been long: a PhD in storage systems at the University of Mainz, building LSM-tree-based append-optimized tables at Greenplum, seven and a half years at Pure Storage working on replication, file systems, and engineering productivity, a year at Databricks building CI infrastructure, and nearly three years as a founding engineer at Augment Code working on AI-powered code completion.

Each role built on the previous one. Each taught me something I didn't know I needed. And now, finally, I'm here.

## What is Napa?

Napa is Google's large-scale data warehousing system. It's not a small project—it powers some of Google's most critical data infrastructure, including analytics for Ads and payments. The system ingests petabytes of data per day and serves billions of queries with sub-second latency.

At its core, Napa is built around log-structured merge trees (LSM-trees) for real-time data ingestion. It maintains materialized views that are updated consistently as new data arrives across multiple data centers. The architecture leverages existing Google infrastructure: Colossus for storage, F1 Query for SQL execution, and Spanner for distributed coordination.

One of Napa's key innovations is the concept of Queryable Timestamp (QT), which provides clients with a live marker for consistent reads across distributed data. This ensures that even as petabytes flow through the system every day, queries see a consistent snapshot of the data.

Napa replaced Mesa, Google's previous data warehousing system, and has been running in production for years. There are academic papers about it ([VLDB 2021](http://www.vldb.org/pvldb/vol14/p2986-sankaranarayanan.pdf), [VLDB 2023](https://www.vldb.org/pvldb/vol16/p3475-sankaranarayanan.pdf)), but much of the system's evolution happens behind closed doors.

## Why I'm Excited

When I built append-optimized tables for Greenplum back in 2013, I implemented an LSM-tree-based design for analytical workloads. It was challenging work—making log-structured storage play nicely with PostgreSQL's MVCC, dealing with distributed compaction, handling edge cases across hundreds of segment nodes.

Napa operates at a completely different scale. The problems aren't just bigger—they're fundamentally different. Consistency across multiple data centers. Petabyte-scale data ingestion. Billions of queries per day. Maintaining materialized views in real-time while the underlying data is constantly changing.

These are exactly the kinds of problems I want to work on.

Beyond the technical challenges, I'm excited to work with the team. The people who built Napa are world-class distributed systems engineers. The codebase has been battle-tested at a scale few systems ever reach. The operational discipline required to keep this system running reliably for Google's critical workloads is immense.

I'm excited to return to database and storage systems while still utilizing my experience with AI infrastructure. My time at Augment Code was transformative—I learned about LLMs, built production AI systems, optimized CUDA kernels, and worked on problems at the frontier of AI for code. But storage systems and databases are where I started. They're problems I keep coming back to. There's something deeply satisfying about building systems that reliably store and retrieve data at scale.

Interestingly, the Napa architecture shares some similarities with the Pure FlashArray architecture. Both are built around distributed, log-structured storage with sophisticated metadata management. This familiarity is reassuring—I'm not starting from scratch.

Last week, I started working on flaky tests. This is like systems software engineer's comfort food. I've done that hundreds of times at Pure Storage. There's something satisfying about tracking down race conditions, understanding timing dependencies, and making tests reliable. It's a perfect way to start getting familiar with the codebase.

## The Journey to Google

Over the years, I've done five full rounds of interviews with Google and passed three. This time, everything lined up.

The first time was in 2008. I was rejected. Looking back, that rejection was correct—I wasn't ready. The gap between where I was and what Google needed was significant. I knew it then, even if I didn't want to admit it.

That rejection stung at the time. But it was the right decision. I needed those years of experience. I needed to ship production systems, debug distributed race conditions, learn from failures, understand how code compounds over years and decades.

The PhD taught me how to think deeply about problems and communicate technical ideas clearly. Greenplum taught me about distributed databases and the reality of legacy codebases. Pure Storage taught me about high-performance systems, engineering discipline, and technical leadership. Databricks taught me about building tooling for rapidly growing engineering organizations. Augment Code taught me how to work at the frontier, how to build customer-facing systems in an entirely new domain, and how to learn quickly when the field is evolving daily.

Each experience was necessary. Not just for getting hired—for being effective once I'm here.

When I look at the Napa codebase now, I can recognize patterns from Greenplum's LSM-tree design, understand the operational challenges from Pure Storage's production systems, appreciate the testing and tooling from my engineering productivity work, and see parallels to the scalability challenges we faced at Augment.

The rejection in 2008 wasn't the end. It was the beginning of a journey that brought me here with the right background and experience.

I'm ready. Not because I'm the same person who applied in 2008, but because I'm not.

---

**Sources:**
- [Napa: Powering Scalable Data Warehousing with Robust Query Performance at Google (VLDB 2021)](http://www.vldb.org/pvldb/vol14/p2986-sankaranarayanan.pdf)
- [Progressive Partitioning for Parallelized Query Execution in Google's Napa (VLDB 2023)](https://www.vldb.org/pvldb/vol16/p3475-sankaranarayanan.pdf)
- [Google Research: Napa Paper](https://research.google/pubs/napa-powering-scalable-data-warehousing-with-robust-query-performance-at-google/)
