---
layout: post
title: Phase 1 | Google Summer of Code
permalink: /gsoc-week-1-report
feature-img: "assets/img/2019-06-03/c-clamp-cash.jpg"
thumbnail: "assets/img/2019-06-03/c-clamp-cash.jpg"
tags: [C++, GSoC, vector<bool>, bits, bitset, ⌨️]
excerpt_separator: <!--more-->
---

It has been about one full week of Google Summer of Code 2019 and so far the direction for a new implementation and replacement for both p0237 and N2050 has been going<!--more--> nicely!

# pWhat, Nwho?

To catch you up on what has been an otherwise completely dead subject since 2008, [`std::tr2::dynamic_bitset` was a proposal by Alisdair Meredith & company](https://wg21.link/N2050) put forth to create a better `std::vector<bool>`. It's impossible to fix `std::vector<bool>` if we don't get a replacement for it, and so getting a replacement is Step 0. Most standard libraries implement `std::tr2::dynamic_bitset`, but the abstraction was a tad limiting and -- like all library proposals that run out of steam and specification maintainers -- ends up being relegated to the "Not Enough Time" graveyard. A later proposal came along for [bit utilities for the header `<bit>` by Dr. Vincent Reverdy & friends, p0236](https://wg21.link/p0237). As covered in my original Google Summer of Code proposal, some standard libraries did not devote cycles to optimizing the bit iterators and friends for their distribution of the standard library. Not only that, one of the devs of libstdc++ has had to reimplement `std::tr2::dynamic_bitset` in several different ways: one with static storage (`std::bitset`), one with dynamic storage (`std::tr2::dynamic bitset`), and then one again that was like `dynamic_bitset` but didn't resize and saved on the required space. The internals of these implementations vary, but not by much: doing this is an exercise in boring and repetitive tedium.

From looking at these implementations, the one in boost, the ones rolled around the world, my own bit twiddling shenanigans and more, I collected some observations.

1. `std::tr2::dynamic_bitset` was only templated on `WordType` and `Allocator`, but essentially mimicked vector.
2. `std::bitset` was templated directly on some size `N`, but could just be seen as a specialized `fixed_vector<N>`, similar to `dynamic_bitset`.
3. The choice of storage scheme is fairly orthogonal to the operations performed on the type: manipulating a container and viewing the result as a series of `0`s and `1`s.

These are not necessarily new ideas or even profound observations.
