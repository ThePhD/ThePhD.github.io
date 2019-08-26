---
layout: post
title: Seizing the Bits of Production - GSoC 2019 Conclusion
permalink: /seize-bits-production-gsoc-2019
feature-img: "assets/img/pexels/hammer-hand-tools-measuring-tape.jpg"
thumbnail: "assets/img/pexels/hammer-hand-tools-measuring-tape.jpg"
tags: [C++, GSoC, bit, ⌨️, 📜]
excerpt_separator: <!--more-->
---

It is finally time.<!--more-->

After 3 months and quite a bit of hard work, I have finally finished the Google Summer of Code program. In it, I have finished an enhanced and improved implementation of [P0237](https://wg21.link/p0237) and [N2050](https://wg21.link/n2050). In my original proposal, I stated that I would implement a range-based `bit_view`, a container adaptor version of `dynamic_bitset`, all of the `bit_iterator`/`bit_reference`/`bit_value` types, and more. Going through the checklist of each phase:


# Phase 1 - bit_iterator, bit_reference, bit_value and bit_view

✔️ 🏆 🎉 - Absolutely covered, with all stretch goals and even MORE met!

This was spoken about [much, much earlier](/std-byte-me-gsoc-2019). We had both `dynamic_bitset<C>` and `bit_view<R>` covered.


# Phase 2 - Iterator Improvements and Algorithms

✔️ 🏆 - Everything, plus most of the stretch goals.

This was covered by 2 previous articles, [here](/proxies-references-gsoc-2019) and [here](/container-adaptors-gsoc-2019). I won't recap absolutely everything, but it was a bit of a struggle getting most of the iterator constraints perfectly okay.

We also added some serious utilities here that cover some of the use cases first brought up after completion of the first phase. [Jonathan Müller mentioned](https://www.reddit.com/r/cpp/comments/c6699n/stdbyte_me_bit_handling_in_a_future_standard/es6vuu0/) that he had some use cases for directly working with a potentially disjoint set of bits. [The `tiny/bit_view`](https://github.com/foonathan/tiny/blob/master/test/bit_view.cpp) class in his library does that. I added support for creating a view of a set of bits that could target precisely bits `[x, y)` by using the `Extents` class portion of the new `bit_view<Range, Extents>`:

```cpp
// 0xFBFF = (MSB) 0b‭1111101111111111‬ (LSB)
std::vector<std::uint32_t> storage{ 0x0000FBFF, 0xFFFFFFFF };
bitsy::bit_view<R, bitsy::static_bit_extents<10, 22>> truncated_view_bits(
	storage.data(), storage.size());
assert(truncated_view_bits.size() == 12);
// 0th bit of biew is 10th bit,
// 10th bit of 0xFBFF is false
assert(truncated_view_bits[0] == bitsy::bit0);
```

I believe taking a class is far more flexible than `span`'s single-numeric-value `extent` type. It also allows for taking a `bitsy::dynamic_bit_extents` type as well, whose size can be calculated entirely at runtime. This was a severe improvement to the API that resulted in covering a much broader selection of use cases while still retaining the same abstraction powers.

Writing the algorithms was simple, really. But the next part is where things got.... interesting.


# Phase 3 - Optimizations

✔️ 🏆 - Everything, plus most of the stretch goals.

Optimizing everything was the hardest part of this Summer of Code, but also the most satisfying. I plan to write an article later with much more detailed information, but the success here is clear as day. There were severe improvements made to the base intrinsics GCC ships contained in libstdc++, since they only work with unsigned types and generally trip over themselves with enumerations. We also identified several places where GCC trips over itself as far as code generation, something that actually made me publish this article today rather than a full week ago (feel free to poke the bear here and see all the zany codegen and constexpr mishaps between MSVC, GCC, and Clang: ). I would like to actually congratulate Clang, which gets the code generation and work done 100% right when you affix the platform:

In all seriousness, I could spend all day writing about this kind of stuff. But, why should I prattle on about algorithmic improvements, intrinsic usage, codegen and the like, when I can simply show some numbers. I re-implemented [Howard Hinnant's performance benchmarks](https://howardhinnant.github.io/onvectorbool.html) for almost all of the algorithms he talked about (sans `std::rotate`). The results are in:

```
----------------------------------------------------------------------
Benchmark                            Time             CPU   Iterations
----------------------------------------------------------------------
noop                             0.000 ns        0.000 ns   1000000000
is_sorted_by_hand                 2807 ns         2762 ns       248889
is_sorted_base                   50573 ns        50223 ns        11200
is_sorted_vector_bool           252820 ns       256696 ns         2800
is_sorted_bitset                202140 ns       199507 ns         3446
is_sorted_itsy_bitsy               839 ns          837 ns       746667
is_sorted_until_by_hand           2812 ns         2825 ns       248889
is_sorted_until_base             51320 ns        51562 ns        10000
is_sorted_until_vector_bool     255426 ns       256696 ns         2800
is_sorted_until_bitset          198101 ns       194972 ns         3446
is_sorted_until_itsy_bitsy         712 ns          715 ns       896000
find_by_hand                      2694 ns         2727 ns       263529
find_base                        23847 ns        24065 ns        29867
find_vector_bool                 89738 ns        89979 ns         7467
find_bitset                      96646 ns        96257 ns         7467
find_itsy_bitsy                   2718 ns         2727 ns       263529
fill_by_hand                       272 ns          273 ns      2635294
fill_base                         2796 ns         2787 ns       263529
fill_vector_bool                   271 ns          270 ns      2488889
fill_bitset                     137941 ns       139509 ns         5600
fill_bitset_smart                  257 ns          257 ns      2488889
fill_itsy_bitsy                    263 ns          268 ns      2800000
fill_itsy_bitsy_smart              257 ns          257 ns      2800000
sized_fill_by_hand                 257 ns          255 ns      2635294
sized_fill_base                   2580 ns         2623 ns       280000
sized_fill_vector_bool          127094 ns       128348 ns         5600
sized_fill_bitset               125951 ns       125552 ns         4978
sized_fill_bitset_smart            251 ns          251 ns      2800000
sized_fill_itsy_bitsy              260 ns          262 ns      2800000
sized_fill_itsy_bitsy_smart        267 ns          268 ns      2800000
equal_by_hand                     1030 ns         1046 ns       746667
equal_memcmp                     0.328 ns        0.312 ns   1000000000
equal_base                       0.318 ns        0.312 ns   1000000000
equal_vector_bool               242570 ns       239955 ns         2800
equal_vector_bool_operator      270586 ns       272770 ns         2635
equal_bitset                     0.000 ns        0.000 ns   1000000000
equal_bitset_operator            0.000 ns        0.000 ns   1000000000
equal_itsy_bitsy                   685 ns          684 ns      1120000
equal_itsy_bitsy_operator        0.321 ns        0.328 ns   1000000000
count_by_hand                     5085 ns         5156 ns       100000
count_base                       46455 ns        46038 ns        11200
count_vector_bool                98858 ns        97656 ns         6400
count_bitset                    114628 ns       114746 ns         6400
count_bitset_smart               10441 ns        10498 ns        64000
count_itsy_bitsy                  5020 ns         5082 ns       144516
count_itsy_bitsy_smart            5083 ns         5022 ns       112000
copy_by_hand                       258 ns          255 ns      2635294
copy_base                         3239 ns         3223 ns       213333
copy_vector_bool                195930 ns       196725 ns         3733
copy_bitset                     147936 ns       150663 ns         4978
copy_bitset_operator               255 ns          257 ns      2800000
copy_itsy_bitsy                    261 ns          261 ns      2635294
copy_itsy_bitsy_operator           264 ns          261 ns      2635294
sized_copy_by_hand                 262 ns          261 ns      2635294
sized_copy_base                   3237 ns         3223 ns       213333
sized_copy_vector_bool          212132 ns       209961 ns         3200
sized_copy_bitset               147954 ns       146484 ns         4480
sized_copy_itsy_bitsy              261 ns          261 ns      2635294
```

There is still performance left on the table here in some cases, but the results are clear: our `bitsy::bit_iterator`s and data structures severely outperform `std::vector<bool>` from current-generation libstdc++ (GCC 9) and either meet or exceed the current algorithmic benchmarks in all categories.

I'm almost certain Alisdair Meredith and Vincent Reverdy would be proud to know that the papers they thought up of wrote over a decade ago, respectively, can enable such great performance. And the best part about this is...


# You Can Have It Too

While I implemented this for libstdc++ and for [itsy/bitsy](https://github.com/ThePhD/itsy_bitsy), the performance is not limited to the standard, or to itsy bitsy. The point is that the iterators are not an implementation-defined, private mess. Nor is it like `std::bitset`, where it has no iterators or `begin()`/`end()` at all. These are well-defined, public interfaces that are going to go into the standard. That means if you write a data structure that works with bits and opt to use `bitsy::bit_iterator` or `__gnu_cxx::bit_iterator`, you get the performance benefits when using `std::equal` and `std::find` and `std::copy` too. So rather than sinking the tiny handful of standard library maintainer time into optimizing a single container, they optimize the algorithm and every gets to benefit.

This also has further downstream consequences: as [pretty as Conor Hoekstra's code](https://www.youtube.com/watch?v=48gV1SNm3WA) is, nobody who cares in performance critical areas is going to use that beautiful solution if it exhibits the `std::vector<bool>` levels of performance. Performance _is_ correctness, and if I can write a disgusting hand-optimized loop and get better performance, you had better believe I'm going to write that disgusting code that would probably make Conor barf.

**But!**

The liberating point about this exploration here is that we can have our beautiful algorithmic cake and develop a powerful algorithmic intuition, and as long as we commit to improving the _algorithm_, we can have our speedy cake too and share it with _everyone_. That means that you, the application developer, can focus on algorithm intuition and reaching Sean Parent levels of "that's a rotate". And the library developers and interested parties can write the optimization in their algorithms and other places. Once. For everyone. For all time. And finally move on to doing more interesting things that twiddling bits, something that should've been a solved problem for 20 years...!

But I digress. There is a lot more to do, but this is it for now.

Remember to optimize your algorithms, not your data structures, and hug your local library developer (with permission).

Catch you later. 💚

P.S.: double underscores are ugly and I hate them with every fiber of my being.
