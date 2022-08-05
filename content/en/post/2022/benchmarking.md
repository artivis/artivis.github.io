---
layout: post
title: A small take on benchmarking software
subtitle: My lib went vroom vroom
date: "2021-10-05T00:00:00Z"

tags: [tutorial, Raspberry Pi, C++]

draft: true
---

In this post we will see how we can benchmark pieces of code and more importantly how we may get more reproducible test runs.

---

## Content

- [What is this all about?](#what-is-this-all-about)

---

## Benchmark you said?

If you are a software developer,
chances are pretty good that,
at a point or another,
you wrote something like the following;

```cpp
auto start = std::chrono::steady_clock::now();

// my piece of code to measure
...

auto end = std::chrono::steady_clock::now();

auto dt = std::chrono::duration_cast<std::chrono::microseconds>(end - start).count();

std::cout << "Elapsed time = " << dt << "[µs]" << std::endl;
```

While I'm a C++ enthusiast,
I'm sure you can relate no matter your poison.
We've all been there.
This may very well be one of the Rite of passage along the software developer journey.

And that totally make sense;
who doesn't want to write better, faster, stronger software?
And to assess that,
we need to measure; to measure properly; to measure a lot.

Alright then let's shove the above snippet in a loop,
and we could keep track of each run duration to compute all kind of statistics,
and we'll need some formatting classes to print all that info nicely in the terminal,
and... Hold on a minute. There has to be a library for that.

## A benchmark library



[//]: # (URLs)

[gbenchmark]: https://github.com/google/benchmark