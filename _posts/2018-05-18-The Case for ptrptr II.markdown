---
layout: post
title: The Case for ptrptr in the Standard Library 2 -- Electric Boogaloo
tags: [C++, ptrptr, ptrptr series, series, performance, benchmarks, 🚌, ⌨️]
excerpt_separator: <!--more-->
---

This is a follow-up to an [earlier blog post]({% post_url 2018-04-15-The Case for ptrptr %}), where we modify the standard library to see if writing a friend implementation gets us the same speed benefits as not writing the C version.

<!--more-->

# The Answer Is Yes.

Similar to the last article on this series, benchmarks were written after we modified libstdc++, libc++, and VC++'s standard library to friend / expose a reference to the internal pointer.

Yes, we were serious enough to modify the standard library!

This is going to be a short post because the answer is exactly as anyone would expect: exposing a reference to the internal pointer and grabbing it directly has the save performance of doing the clever, undefined-behavior aliasing trick as before:

![Local ptrptr benchmarks.](https://raw.githubusercontent.com/ThePhD/ptrptr/master/benchmark_results/local.png)

![Reset ptrptr benchmarks.](https://raw.githubusercontent.com/ThePhD/ptrptr/master/benchmark_results/reset.png)

This definitely means that by having the simple `ptrptr` implementation in the standard library, where the smart pointers can safely friend the implementation without leaking its reference to the world, is ideal.

So, it's time to write a paper!
