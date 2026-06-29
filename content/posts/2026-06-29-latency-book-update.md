---
title: "Book Update: Latency, and Why Low-Level Knowledge Is Back"
date: 2026-06-29
---

Back in 2020, I wrote a [list of books for new college grads](https://dmeister.github.io/blog/2020/09/07/books/) starting a career as a software engineer. That list still mostly holds up, but it is time for an update, anchored by one standout: ["Latency: Reduce Delay in Software Systems"](https://www.manning.com/books/latency) by Pekka Enberg, published by Manning in late 2025.

## Why an Update Now

The 2020 list was about becoming a software engineer in general: contracts, clean code, design patterns, the craft. It still applies. But something has shifted since then. For a decade, the industry could mostly assume that compute would keep getting cheaper and more abundant, that the next CPU generation would paper over inefficiency. That assumption is under more strain now. AI workloads consume compute at a pace hardware can't comfortably keep up with, and the free performance lunch from new silicon has been slowing for a while. GPUs and accelerators are scarce and expensive. Distributed systems have to do more with the resources they have, not just more resources.

That makes a different kind of knowledge valuable again: understanding where time actually goes in a system, from cache misses up through synchronization to network round trips across data centers. Not as a niche specialty, but as baseline literacy for anyone building serious infrastructure.

## Why "Latency" Stands Out

Most books in this space pick a layer and stay there. You get a book about CPU caches and memory hierarchies, or a book about lock-free algorithms, or a book about distributed systems consistency. "Latency" is unusual because it deliberately moves across all of them and shows how the same underlying ideas, Little's Law, queueing, contention, show up differently at each layer.

The book walks from modeling and measuring latency, through colocation, replication, partitioning, and caching, into eliminating work and wait-free synchronization, and finally into asynchronous and predictive techniques. Pekka Enberg's background, Linux kernel work plus the Scylla and Turso databases, shows in how concretely the book treats each layer. The examples are in Rust, but the ideas transfer directly to C++ or any systems language.

This is exactly the gap I felt in 2020. "Effective Modern C++" goes deep on language-level mechanics. The GoF book and "Code Complete" are about structure and process. None of them connect a cache-line bounce to a lock-free queue to a retry storm in a distributed system. "Latency" does, in one coherent narrative.

## Why It Resonates With My Experience

At Pure Storage, we spent years on a flash storage system where latency was the product, not a side effect. Queueing behavior under load, the cost of synchronization on the hot path, the gap between single-node and distributed tail latency, all of that was daily work, learned the hard way through production incidents and benchmarks. At Napa, distributed systems latency shows up again, just at a different scale and with different consistency tradeoffs.

A book that ties these threads together into a single mental model would have saved me a lot of scattered learning, picked up the hard way across storage systems and distributed databases instead of from one coherent source.

## The Rest of the Updated List

The 2020 list does not need a rewrite. "Effective Modern C++", "Effective Java", "Code Complete 2", and the others are still solid foundations for writing correct, maintainable code. What changed is the layer above that: once you can write good code, "Latency" is the book I would now hand to an engineer who needs to make that code fast, and keep it fast as it moves from a single machine to a distributed system. In a hardware-constrained era, that is no longer a specialist skill. It is core engineering.
