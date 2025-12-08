---
title: "From Storage to Engineering Productivity to AI for Code"
date: 2026-01-03
aliases:
  - /blog/posts/2026-01-03-ai-for-code-journey/
---

After leaving Pure Storage in 2021, I remained in the Engineering Productivity space, joining Databricks to work on internal tooling. One of my main focuses was [Runbot, Databricks' CI solution](https://www.databricks.com/blog/2021/10/14/developing-databricks-runbot-ci-solution.html). The work was challenging and impactful, helping a rapidly growing engineering organization maintain velocity and quality.

However, after just over a year, I received an unexpected call from a former colleague: "Hey, we're starting something new. It's AI for Code."

## The Beginning: Four People in a Room

I joined [Augment Code](https://www.augmentcode.com/) as a founding engineer. We were four people in a small room. This was before ChatGPT, before LLMs became mainstream, when "AI for code" was still a niche concept that required explanation. I primarily focused on the production system: LLM model serving, code indexing, and retrieval, but also operations, monitoring, deployment, security, and so on.

In hindsight, there are a couple of highlights of this time:

## The Importance of Latency

One of the earliest lessons we learned was the critical importance of latency. A good code completion that appears after 300ms will be accepted by developers. The same completion arriving after 700ms? Too late. The developer has already moved on, typed the next few characters, and your suggestion is now irrelevant.

This aspect was actually a huge part which drove me to join Augment. I like working on Engineering Productivity and always will. I like working on Systems (Performance and Latency critical) and always will. Augment was a place where I could work on both at the same time. After a few years in Eng Productivity, I wanted to work more on Systems.

## Security From Early On

From early on, the system was designed with security in mind. When you're asking customers to trust you with their codebase, it's a significant ask, to be honest. As I discussed on the [YSecurity podcast](https://thesecuritypodcastofsiliconvalley.podbean.com/e/inside-augment-code-why-security-starts-at-line-one-with-dirk-meister/), I strongly believe security and trust is hard to bolt on later.

I authored (and co-authored) two blog posts about our approach in detail:
- [Securing the Code That Writes Code: A Look Inside Our AI Platform](https://www.augmentcode.com/blog/securing-the-code-that-writes-code-a-look-inside-our-ai-platform)
- [A Real-Time Index for Your Codebase: Secure, Personal, Scalable](https://www.augmentcode.com/blog/a-real-time-index-for-your-codebase-secure-personal-scalable)

## ChatGPT Changes Everything

A few months after I joined Augment Code, ChatGPT launched and changed the game overnight. LLMs went from niche technology to mainstream phenomenon. The transformation was remarkable. It was surreal working on an area that had just exploded in relevance and importance. Most of my career was in storage, which is not an area like that.

## Technical Challenges at Scale

The technical problems were fascinating. We built a custom inference stack for our embedding models, optimizing GPU utilization beyond what generic cloud APIs could provide. The system needed to process thousands of files per second while maintaining responsive personal index updates. The largest customers could have tens of thousands of engineers, with 100,000+ or more files in their repositories.

I am proud of what we built in the Augment backend.

## Learning at the Frontier

The learning curve was constant. I found myself:

- Deep-diving into "Attention Is All You Need" and decoder-only transformer architectures, staring at tensor outputs, trying to understand what was a bug and what was numerical instability
- Working on inference optimization with some of the leading researchers in the field
- Being in the room early enough to design architectures that would hopefully stand the test of time. Code can always be fixed, but architecture is sticky.
- Prototyping new ideas, making mistakes, fixing mistakes, iterating rapidly
- Going on-call and getting paged for production systems, which I had done at Databricks, but only for internal tools. This was different. These were customer-facing systems. Paying customers. Enterprise customers.

There were days where I was writing CUDA kernels and TypeScript for the VS Code extension.

Being a founding engineer at a startup in such a transformative space meant wearing many hats and learning across the entire stack: From LLMs, CUDA to Kubernetes infrastructure, from security to user experience. It was a special experience that I learned a lot from.

## The Next Chapter

I recently made the decision to leave Augment. My last day was December 5, 2025. I'm grateful for the journey, the people I've worked with, and the problems I've had the opportunity to solve. And I'm also excited about what comes next.
