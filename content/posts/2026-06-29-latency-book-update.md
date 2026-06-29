---
title: "Books Update: Latency, the Missing README, and Crafting Interpreters"
date: 2026-06-29
---

Back in 2020, I wrote a [list of books for new college grads](https://dmeister.github.io/blog/2020/09/07/books/) starting a career as a software engineer. That list still mostly holds up, but it is time for an update with three additions: ["Latency: Reduce Delay in Software Systems"](https://www.manning.com/books/latency) by Pekka Enberg (Manning, late 2025), which is timely; ["The Missing README"](https://nostarch.com/missing-readme) by Chris Riccomini and Dmitriy Ryaboy (No Starch Press), which is overdue; and ["Crafting Interpreters"](https://craftinginterpreters.com/) by Robert Nystrom, which is just for fun.

## Why an Update Now

The 2020 list was about becoming a software engineer in general: contracts, clean code, design patterns, the craft. It still applies. But something has shifted since then. For a decade, the industry could mostly assume that compute would keep getting cheaper and more abundant, that the next CPU generation would paper over inefficiency. That assumption is under more strain now. AI workloads consume compute at a pace hardware can't comfortably keep up with, and the free performance lunch from new silicon has been slowing for a while. GPUs and accelerators are scarce and expensive. Distributed systems have to do more with the resources they have, not just more resources.

That makes a different kind of knowledge valuable again: understanding where time actually goes in a system, from cache misses up through synchronization to network round trips across data centers. Not as a niche specialty, but as baseline literacy for anyone building serious infrastructure.

## Why "Latency" Stands Out

Most books in this space pick a layer and stay there. You get a book about CPU caches and memory hierarchies, or a book about lock-free algorithms, or a book about distributed systems consistency. "Latency" is unusual because it deliberately moves across all of them and shows how the same underlying ideas, Little's Law, queueing, contention, show up differently at each layer.

The book walks from modeling and measuring latency, through colocation, replication, partitioning, and caching, into eliminating work and wait-free synchronization, and finally into asynchronous and predictive techniques. Pekka Enberg's background, Linux kernel work plus the Scylla and Turso databases, shows in how concretely the book treats each layer. The examples are in Rust, but the ideas transfer directly to C++ or any systems language.

This is exactly the gap I felt in 2020. "Effective Modern C++" goes deep on language-level mechanics. The GoF book and "Code Complete" are about structure and process. None of them connect a cache-line bounce to a lock-free queue to a retry storm in a distributed system. "Latency" does, in one coherent narrative.

## Why It Resonates With My Experience

At Pure Storage, we spent years on a flash storage system where latency was the product, not a side effect. Queueing behavior under load, the cost of synchronization on the hot path, the gap between single-node and distributed tail latency, all of that was daily work, learned the hard way through production incidents and benchmarks.

At Augment Code, we built our backend in Rust, and vector search was one of the most latency-sensitive parts of it: every completion and chat request depended on it, on the critical path, at low latency. Last quarter I got to spend real time on latency improvements there, and while not everything about that work was rosy, that part of it was genuinely fun. Exactly the kind of work this book gives you a vocabulary for.

At Napa, distributed systems latency shows up again, just at a different scale and with different consistency tradeoffs.

A book that ties these threads together into a single mental model would have saved me a lot of scattered learning, picked up the hard way across storage systems and distributed databases instead of from one coherent source.

## The One Actually Written for New Grads

Looking back at the 2020 list with fresh eyes, none of those books are really new-grad specific. "Effective Modern C++", "Code Complete", the GoF book, they are good engineering books that any engineer at any level could pick up. They are not about the actual transition from student to employee.

["The Missing README"](https://nostarch.com/missing-readme) by Chris Riccomini and Dmitriy Ryaboy is. It covers the things nobody teaches in school and nobody quite remembers to explain on the job: how to work in an existing codebase instead of a green field, how technical debt actually accumulates and gets paid down, what a good code review looks like from both sides, how to safely ship and roll back, and what to do when you are on call and something is on fire. If I had to hand a new college grad exactly one book on day one, this is closer to the right one than anything on my 2020 list.

## The Fun One: Crafting Interpreters

["Crafting Interpreters"](https://craftinginterpreters.com/) by Robert Nystrom is the odd one out here: less practical, more fun. It is the most enjoyable CS book I have read in a decade.

The book builds two real, complete interpreters for a small language called Lox: a tree-walking interpreter in Java, then a bytecode virtual machine in C. No hand-waving, no "left as an exercise", every line of both implementations is in the book and explained. Nystrom writes with a wit that is rare in technical books without ever being sloppy about the actual computer science: the explanations of parsing, scoping, closures, and garbage collection are correct and precise, just delivered with a sense of humor. It manages to be a serious compilers book and an engaging read at the same time, which is a hard combination to pull off.

And it is not purely recreational. Knowing how to write a small recursive descent parser by hand is a genuinely useful skill that shows up more often than people expect, in config languages, query languages, internal DSLs, and the occasional debugging session staring at someone else's grammar.

## The Rest of the Updated List

The 2020 list does not need a rewrite. "Effective Modern C++", "Effective Java", "Code Complete 2", and the others are still solid foundations for writing correct, maintainable code. What changed is the layer above and around that: "Latency" is the book I would now hand to an engineer who needs to make that code fast and keep it fast as it moves from a single machine to a distributed system, "The Missing README" is the one for the first weeks on the job, and "Crafting Interpreters" is the one to read for fun on a weekend, with a useful skill as a side effect. In a hardware-constrained era, the latency knowledge especially is no longer a specialist skill. It is core engineering.
