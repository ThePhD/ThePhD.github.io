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

This was spoken about [much, much earlier](/std-byte-me-gsoc-2019). We had both `dynamic_bitset<C>` and `bit_view<R>` covered, which pulled in an implementation of `bit_iterator`, `bit_reference`, and `bit_value`. Unfortunately, during this time we also had `bit_span`, which was a bad name for a type. The original idea was having a mutable versus immutable split for each container in separate types. That was a mistake and it was far better to just let the `const`-ness of the range or container flow naturally into the class's interface and compile-time prevent things. There's still a `bit_span` in the final product, but it's just an alias for `template <typename T> using bit_span = bit_view<span<T>>;`.

The only other change here is that our `bit_reference` type is a bit more involved, taking a `ReferenceType` and a `MaskType` rather than just the default `bit_reference<T>` of the original proposal. This allows for a greater degree of control over how the reference shakes out, including potential uses for `std::reference<>` types being used in `bit_reference` successfully. (Currently, this scenario is not tested, but we _do_ allow for a top-level `bit_view<std::reference<std::vector>>` behavior!)


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

Most people would wonder why I didn't take the stance of using a number directly like when they made `std::span<T, N>`. I believe taking a class template parameter is far more flexible than `span`'s single numeric `extent` value. It also allows for taking a `bitsy::dynamic_bit_extents` type as well, whose size can be calculated entirely at runtime. The default is a `bitsy::word_bit_extents`, which simply spans from 0 to the size of the container (which is, e.g., the extent of the words of the range). This was a severe improvement to the API that resulted in covering a much broader selection of use cases while still retaining the same abstraction powers.

Writing the algorithms was simple, really. It turns out most of the algorithms in C++ aren't too hard when you sit down with them and read the requirements and listen to conference talks and read text books about them.

But the next part? Getting things fast is where things got.... interesting.


# Phase 3 - Optimizations

✔️ 🏆 - Everything, plus most of the stretch goals.

Optimizing everything was the hardest part of this Summer of Code, but also the most satisfying. I plan to write an article later with much more detailed information, but the success here is clear as day. There were severe improvements made to the algorithms GCC ships in libstdc++, since they only work with unsigned types and generally trip over themselves with enumerations. We also identified several places where GCC trips over itself as far as code generation, something that actually made me publish this article today rather than a full week ago (feel free to poke the bear here and see all the zany codegen and constexpr mishaps between MSVC, GCC, and Clang: ). I would like to actually congratulate Clang, which gets the code generation and work done 100% right when you affix the platform:

In all seriousness, I could spend all day writing about this kind of stuff. But, why should I prattle on about algorithmic improvements, intrinsic usage, codegen and the like, when I can simply show some numbers. I re-implemented [Howard Hinnant's performance benchmarks](https://howardhinnant.github.io/onvectorbool.html) for almost all of the algorithms he talked about (sans `std::rotate`). The results are in:

```
----------------------------------------------------------------------
Benchmark                            Time             CPU   Iterations
----------------------------------------------------------------------
noop                             0.000 ns        0.000 ns   1000000000
is_sorted_by_hand                 2842 ns         2888 ns       248889
is_sorted_base                   56290 ns        57199 ns        11200
is_sorted_vector_bool           273079 ns       278308 ns         2358
is_sorted_bitset                192486 ns       194972 ns         3446
is_sorted_itsy_bitsy               817 ns          820 ns       896000
is_sorted_until_by_hand           2812 ns         2825 ns       248889
is_sorted_until_base             48404 ns        48652 ns        14452
is_sorted_until_vector_bool     251334 ns       251116 ns         2800
is_sorted_until_bitset          187284 ns       185904 ns         3446
is_sorted_until_itsy_bitsy         679 ns          680 ns       896000
find_by_hand                      2443 ns         2455 ns       280000
find_base                        22696 ns        22949 ns        32000
find_vector_bool                 84364 ns        85449 ns         8960
find_bitset                      90512 ns        89979 ns         7467
find_itsy_bitsy                   2422 ns         2400 ns       280000
fill_by_hand                       256 ns          255 ns      2635294
fill_base                         2533 ns         2511 ns       280000
fill_vector_bool                   254 ns          257 ns      2800000
fill_bitset                     126316 ns       125552 ns         4978
fill_bitset_smart                  255 ns          257 ns      2800000
fill_itsy_bitsy                    259 ns          262 ns      2800000
fill_itsy_bitsy_smart              254 ns          255 ns      2635294
sized_fill_by_hand                 252 ns          251 ns      2800000
sized_fill_base                   2567 ns         2567 ns       280000
sized_fill_vector_bool          113136 ns       112305 ns         6400
sized_fill_bitset               124528 ns       125558 ns         5600
sized_fill_bitset_smart            254 ns          251 ns      2800000
sized_fill_itsy_bitsy              257 ns          257 ns      2800000
sized_fill_itsy_bitsy_smart        256 ns          257 ns      2800000
equal_by_hand                     1020 ns         1025 ns       640000
equal_memcmp                     0.321 ns        0.312 ns   1000000000
equal_base                       0.323 ns        0.328 ns   1000000000
equal_vector_bool               242796 ns       239955 ns         2800
equal_vector_bool_operator      256364 ns       254981 ns         2635
equal_bitset                     0.000 ns        0.000 ns   1000000000
equal_bitset_operator            0.000 ns        0.000 ns   1000000000
equal_itsy_bitsy                   518 ns          516 ns      1000000
equal_itsy_bitsy_operator          513 ns          516 ns      1120000
count_by_hand                     5606 ns         5625 ns       100000
count_base                       46712 ns        46490 ns        14452
count_vector_bool                98263 ns        97656 ns         6400
count_bitset                    111041 ns       109863 ns         6400
count_bitset_smart               11085 ns        10986 ns        64000
count_itsy_bitsy                  5509 ns         5580 ns       112000
count_itsy_bitsy_smart            5500 ns         5625 ns       100000
copy_by_hand                       259 ns          255 ns      2635294
copy_base                         3202 ns         3209 ns       224000
copy_vector_bool                192254 ns       192540 ns         3733
copy_bitset                     146175 ns       146484 ns         4480
copy_bitset_operator               262 ns          261 ns      2635294
copy_itsy_bitsy                    258 ns          262 ns      2800000
copy_itsy_bitsy_operator           259 ns          255 ns      2635294
sized_copy_by_hand                 259 ns          255 ns      2635294
sized_copy_base                   3258 ns         3278 ns       224000
sized_copy_vector_bool          220069 ns       219727 ns         3200
sized_copy_bitset               151193 ns       149972 ns         4480
sized_copy_itsy_bitsy              269 ns          267 ns      2635294
```

There is still performance left on the table here in some cases, but the results are clear: our `bitsy::bit_iterator`s and data structures severely outperform `std::vector<bool>` from current-generation libstdc++ (GCC 9) and either meet or exceed the current algorithmic benchmarks in all categories.

Here are the results for VC++ from Visual Studio 16.2.3:

```
----------------------------------------------------------------------
Benchmark                            Time             CPU   Iterations
----------------------------------------------------------------------
sized_copy_by_hand                 554 ns          558 ns      1120000
sized_copy_base                   2670 ns         2668 ns       263529
sized_copy_vector_bool          201823 ns       204041 ns         3446
sized_copy_bitset               189604 ns       188354 ns         3733
sized_copy_itsy_bitsy              190 ns          188 ns      3733333
copy_by_hand                       606 ns          614 ns      1120000
copy_base                         2431 ns         2431 ns       263529
copy_vector_bool                244844 ns       245857 ns         2987
copy_bitset                     171791 ns       172631 ns         4073
copy_bitset_operator               147 ns          146 ns      4480000
copy_itsy_bitsy                    170 ns          167 ns      3733333
copy_itsy_bitsy_operator           145 ns          148 ns      4977778
count_by_hand                     2728 ns         2727 ns       263529
count_base                       67899 ns        65569 ns        11200
count_vector_bool               132797 ns       131830 ns         4978
count_bitset                     98924 ns        97656 ns         6400
count_bitset_smart                5299 ns         5301 ns       112000
count_itsy_bitsy                  2563 ns         2550 ns       263529
count_itsy_bitsy_smart            2601 ns         2609 ns       263529
equal_by_hand                     1023 ns         1001 ns       640000
equal_memcmp                       518 ns          516 ns      1120000
equal_base                       65633 ns        65569 ns        11200
equal_vector_bool               230969 ns       230164 ns         2987
equal_vector_bool_operator         525 ns          531 ns      1000000
equal_bitset                    146907 ns       146484 ns         4480
equal_bitset_operator              528 ns          531 ns      1000000
equal_itsy_bitsy                   532 ns          531 ns      1000000
equal_itsy_bitsy_operator          527 ns          516 ns      1120000
sized_fill_by_hand                 515 ns          516 ns      1120000
sized_fill_base                   1805 ns         1800 ns       373333
sized_fill_vector_bool          113288 ns       111607 ns         5600
sized_fill_bitset               145251 ns       144385 ns         4978
sized_fill_bitset_smart            138 ns          138 ns      4977778
sized_fill_itsy_bitsy              162 ns          164 ns      4480000
sized_fill_itsy_bitsy_smart        167 ns          167 ns      3733333
fill_by_hand                       504 ns          502 ns      1120000
fill_base                         1824 ns         1842 ns       407273
fill_vector_bool                149444 ns       149613 ns         4073
fill_bitset                     145098 ns       147524 ns         4978
fill_bitset_smart                  147 ns          148 ns      4977778
fill_itsy_bitsy                    566 ns          572 ns      1120000
fill_itsy_bitsy_smart              163 ns          161 ns      4072727
find_by_hand                     64005 ns        64174 ns        11200
find_base                        45115 ns        45516 ns        15448
find_vector_bool                114227 ns       114397 ns         5600
find_bitset                      78854 ns        77424 ns         7467
find_itsy_bitsy                  64992 ns        64523 ns         8960
is_sorted_until_by_hand          49151 ns        48828 ns        11200
is_sorted_until_base             65651 ns        65569 ns        11200
is_sorted_until_vector_bool     231133 ns       234375 ns         3200
is_sorted_until_bitset          161046 ns       163923 ns         4480
is_sorted_until_itsy_bitsy        1459 ns         1475 ns       497778
is_sorted_by_hand                52399 ns        51618 ns        11200
is_sorted_base                   68395 ns        68011 ns         8960
is_sorted_vector_bool           236765 ns       235395 ns         2987
is_sorted_bitset                162437 ns       163923 ns         4480
is_sorted_itsy_bitsy              1420 ns         1444 ns       497778
noop                             0.325 ns        0.328 ns   1000000000
```

Note that the performance here might not be as great. In order to preserve `constexpr`, we could not use intrinsics in VC++ because none of their intrinsics are constexpr. Their compiler does not have `std::is_constant_evaluated()` in any form, which makes it impossible to use low-level, fast functions in MSVC for the algorithms and other places without giving `constexpr` the guillotine. Hopefully, the situation on that front will improve and we will get better constexpr-related goodies in the releases to come.

I'm almost certain Alisdair Meredith and Vincent Reverdy would be proud to know that the papers they thought up of and 13 and 3 years ago, respectively, can enable such great performance. And the best part about this is...


# You Can Have It Too

While I implemented this for [libstdc++](https://gcc.gnu.org/ml/gcc-patches/2019-08/msg01771.html) and for [itsy.bitsy](https://github.com/ThePhD/itsy_bitsy), the performance is not limited to the standard, or to itsy bitsy. The point is that the iterators are not an implementation-defined, private mess. Nor is it like `std::bitset`, where it has no iterators or `begin()`/`end()` at all. These are well-defined, public interfaces that are going to go into the standard. That means if you write a data structure that works with bits and opt to use `bitsy::bit_iterator` or `__gnu_cxx::bit_iterator`, you get the performance benefits when using `std::equal` and `std::find` and `std::copy` too. So rather than sinking the tiny handful of standard library maintainer time into optimizing a single container, they optimize the algorithm and everyone gets to benefit.

Note that there are even more optimizations to perform here: there are plenty of optimization opportunities when someone copies a `bit_iterator<std::byte*>` to, say a `bit_iterator<std::uint64_t*>`. The current overloads don't handle such a case (because it is a difficult case to handle generically and with all due speed). Being able to tackle that in a far more generic fashion will be important to the continued longevity of making the basic abstractions we use faster and better than before.

This also has further downstream consequences: as [pretty as Conor Hoekstra's code](https://www.youtube.com/watch?v=48gV1SNm3WA) is, nobody who cares in performance critical areas is going to use that beautiful solution if it exhibits the `std::vector<bool>` levels of performance. Performance _is_ correctness, and if I can write a disgusting hand-optimized loop and get better performance, you had better believe I'm going to write that disgusting code that would probably make Conor very sad.

**But!**

The liberating point about this exploration here is that we can have our beautiful algorithmic cake and develop a powerful algorithmic intuition, and -- as long as we commit to improving the _algorithm_ -- we can have our speedy cake too and share it with _everyone_. That means that you, the application developer, can focus on algorithm intuition and reaching Sean Parent levels of "that's a rotate". And the library developers and interested parties can write the optimization in their algorithms and other places. Once. For everyone. For all time. And finally move on to doing more interesting things that twiddling bits, something that should've been a solved problem for 20 years! Algorithms no longer need to be for the Computer Science Gods and the Geniuses: we, the proletariat, can seize the means of performance from the bourgeoisie and the regular plebeian developer can write `std::copy` and `std::equal` and have it be exactly as fast as intended...!

But I digress. There is a lot more to do, but this is it for now. The GCC patch can be reviewed but albeit it's likely going to be rejected because there's some work I need to do before hand to make sure the tests can work in an isolated GCC. Most notably, I need to implement a `std::span` for libstdc++ and I need a `std::ranges::subrange` type. Then all the tests will work directly.

Remember to optimize your algorithms, not your data structures, and hug your local library developer (with permission).

Catch you later. 💚

<sup><sup>P.S.: double underscores are ugly and I hate them with every fiber of my being.</sup></sup>
