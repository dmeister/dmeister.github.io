---
title: "Joining Google: 18 Years in the Making"
date: 2026-01-18
---

On January 5, I started at Google as a Software Engineer on the Napa database team.

[Napa](https://www.vldb.org/pvldb/vol14/p2986-sankaranarayanan.pdf) is Google's internal large-scale data warehousing system built around log-structured merge trees ([LSM-trees](https://dsf.berkeley.edu/cs286/papers/lsm-acta1996.pdf)) for real-time data ingestion. Its key feature is maintaining materialized views that stay consistent as new data arrives across multiple data centers, supporting queries with sub-second latency on petabyte-scale datasets.

# Why I'm Excited

Returning to database and storage systems feels right. My time in Engineering Productivity at Databricks and at [Augment Code](https://www.augmentcode.com/) was inspiring, but storage and databases are where I started, problems I keep coming back to. They need to work, are challenging to get right, and have strong foundations in both research and engineering.

Last week, I got my first starter task: hunting down flaky tests. This is a systems software engineer's comfort food. I've done that hundreds of times at Pure Storage.

This is also the first time I'm not joining a startup. Whether Databricks in 2021 was still a startup can be debated, but Google definitely isn't. At Augment, I was employee 3, still working from the [Sutter Hill Ventures](https://shv.com/) office in Palo Alto. In many ways, small startups feel more natural to me, but after Augment I wanted something different: navigating a massive organization, understanding systems people have worked on for more than a decade, working at unprecedented scale.

This is what Gemini generated for me:

<img src="/post_img/2026-01-18.png" alt="Pixel art of engineer at workstation with multiple monitors" style="max-width: 500px; display: block; margin: 1em auto;">

## The Journey to Google

It wasn't a direct path. The first time I applied was in 2008 during my Master's studies. I interviewed at Google Munich, which at the time was a very small office with only a dozen or so engineers. I was rejected. Looking back, I wasn't ready.

Over the years, I've done five full rounds of interviews with Google and passed three. This time, everything lined up.

Each step in my career prepared me for the next: the PhD taught me to think deeply about technical problems; Greenplum taught me distributed databases and legacy codebases; Pure Storage taught me high-performance systems, engineering discipline, and technical leadership. The genius of the FA architecture and the engineering talent during my early years at Pure are something I will cherish forever. At Databricks I learned about working in a sprawling, growing organization. Augment Code taught me AI infrastructure and working with LLMs before it was "cool".

When I look at the Napa codebase now, I recognize patterns from Pure Storage's ["Pyramid"](https://dl.acm.org/doi/epdf/10.1145/2723372.2742798), see operational aspects from Databricks and Augment, and appreciate testing practices from my engineering productivity work.

The 2008 rejection wasn't the end. It was the start of a journey that brought me here ready.
