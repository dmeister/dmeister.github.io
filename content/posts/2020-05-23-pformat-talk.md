---
title: A follow up to the pformat talk for Pure Storage Prague
date: 2020-05-23
aliases:
  - /blog/posts/2020-05-23-pformat-talk/
---

Last year, I got interested in the topic of compile-time programming in C++ and in December I published the format library at [github.com/dmeister/pformat](https://github.com/dmeister/pformat). It is clear to me that the library is not and likely never will be production-ready. It was an experiment if compile-time string format checking is possible. In fact, it is possible and has been done many times before. I just didn’t knew that. Similar things have been done by [fmt](https://www.zverovich.net/2017/11/05/compile-time-format-strings.html) or [pmark/format](https://github.com/mpark/format).

On the other hand, the library didn’t appear to be completely off the mark and it is small enough to explain in 30 minutes or so. So, Pure Prague asked me to give a presentation about it and here we go.

Long story short, last Tuesday, I have given a public presentation for our Prague office about compile-time programming in C++.
I used the pformat library as an example and showed constexpr functions, if constexpr and template-meta programming and how they are used in the library.

After the talk, there were pointed questions by a knowledgeable person (“knowledgeable” is a grave understatement, but I am not sure if the person asking would like to see their name mentioned in this blog post) about the compile-time impact. Before, I was quite cavalier about it. The compile times at Pure are quite high and the problem is solved with hardware. Our build system “pb” is backed by a significant compilation cluster, so I mostly forget about it.

However, the point is taken. I should do better. Esp. before giving presentations about
the topic. The library is certainly not optimized for it. Saying that it overuses template meta programming is fair. So, first things first: how bad is it?

Here are the results when pformat is added to the bloat-test of fmtlib ([Github fork](https://github.com/dmeister/format-benchmark)) on my 1.1 GHz Dual-Core Intel Core m3 Macbook:

Method        |Compile Time, s |Executable size, KiB |Stripped size, KiB
--------------|----------------|---------------------|-------------------
printf        |            5.3 |                  33 |                30
printf+string |           47.6 |                  33 |                30
IOStreams     |           95.1 |                  63 |                59
fmt           |           74.9 |                 133 |               114
compiled_fmt  |           66.0 |                 133 |               114
tinyformat    |          136.3 |                 113 |               105
Boost Format  |          353.5 |                 230 |               200
Folly Format  |          380.4 |                 103 |                90
stb_sprintf   |            6.9 |                  62 |                58
pformat       |          130.0 |                  82 |                78

The pformat library has very high compilation times. The test builds 100 translation unit tech printing
5 formats with a mix of integers and strings with 1 to 5 parameters each. That is not
a lot of functionality.
The compilation times some of other libraries are surprisingly high, too.

So, it is clear: Reducing compilation times of pformat is the next task. There is still
a lot to learn and understand the tradeoffs.
