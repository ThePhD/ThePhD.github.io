---
layout: post
title: std::byte me! - Bit Handling in a Future Standard Library
permalink: /std-byte-me-gsoc-2019
feature-img: "assets/img/2019-06-27/cat bite me.jpg"
thumbnail: "assets/img/2019-06-27/cat bite me.jpg"
tags: [C++, GSoC, bit, ⌨️, 📜]
excerpt_separator: <!--more-->
---

In an earlier post, I talked about getting into Summer of Code. Well, phase 1 of my Summer of Code project is now complete! The implementation of Dr. Vincent Reverdy's [P0237 - bit manupulation library](https://wg21.link/p0237) is done alongside some new views / containers, but that doesn't mean there isn't quite a few things left<!--more--> to discuss and a wide variety of additional support that needs to be worked out! But first...



# Motivation: std::vector of bool is kind of bad

`std::vector<bool>` is mostly bad because of its name. It's not a vector, it's definitely not a vector of bool, it does not give you `bool&` objects in memory you can reference directly, and more. `boost::dynamic_bitset` changes the name out, enhances the interface, but then fails to be customizable in any truly meaningful fashion by the programmer. Changing the block size and the allocator are small consolation prizes in the world of bits. Interested parties want to represent billions of hash keys, many with 128 bits or less of stored information: choosing a heap-allocated, dynamically resizable container that also bookkeeps its own size is very much not desired in these spaces. It's not just researchers or big companies: libstdc++ rolls a handful of its own internal data structures of messing with bits _besides_ what `std::vector<bool>` has to offer. One views a span of bits and allows someone to manipulate them but not grow / resize, for example: that is very hard to capture without overhead in the `boost::dynamic_bitset` model, and impossible in the world of `std::vector<bool>`.

So, we need more flexibility. Unfortunately, implementing all of `unresizable_bit_span` (currently not widely available), `resizable_bit_array` (e.g. `vector<bool>`), `fixed_bit_array` (e.g. `std::bitset<N>`),  and even `ordered_bits` ([e.g. Dr. Halbersma's `xstd::bit_set`](https://github.com/rhalbersma/bit_set#the-current-bitset-landscape)). It would be a waste of everyone's time if the solution was to add One More Container With Slightly Varying Storage Properties to the growing amalgamation of bit types, let alone in the standard.

Therefore, we need to encapsulate all of that.



# Enter Summer of Code 2019 and Vincent Reverdy

The goal of this Summer of Code 2019 -- under the GNU Compiler Collection and Free Software Foundation -- is to essentially implement both the base utilities present in [P0237](https://wg21/.link/p0237) (with modifications to fit the domain), and provide useful and generic bit containers. The most powerful realization is that there's no need to bundle a specific storage scheme with the new containers and views: they can be written as generic adaptors over the concepts of what a `Range`, `View`, and `SequenceContainer` are. This means we define 3 new views/container adaptors that take an existing view / storage and simply view it as a sequence of bits!

I will also be working closely with the original [P0237](https://wg21/.link/p0237) author, Vincent Reverdy, on an implementation that will exist outside of what may end up in libstdc++. If all of this works successfully, I will move to put it in libc++ too if time permits, and the standard as well, though that might be on my own time and dollar.


### Container, Range, View... Span?

When I originally handed in the proposal for GSoC 2019 and then modified the submission to change my goals, I had in mind -- at most -- 2 containers. `bitset_view`, and `dynamic_bitset`. As I developed the data structures for `bitset_view` and `dynamic_bitset`, I surveyed a few uses of code in libstdc++ and also took a look at a few different applications of `std::bitset` and `boost::dynamic_bitset`. A large majority of the uses needed either a const-like, immutable view of bits or a non-resizing, mutable view. The rest of the cases were covered by the typical `boost::dynamic_bitset` needs. Therefore, the design space and target goals have evolved, which has led to the implementation of 3 separate ranges / views / containers:

- `bit_view`
- `bit_span`
- `dynamic_bitset`

The names might need some bikeshedding, but that's alright because nothing is set in stone yet. Things can be improved, since this is not being cast in Standard Steel just yet. We need a place for these types, and so...


# A new library: itsy_bitsy

`itsy_bitsy` -- namespace `bitsy::` -- is a ~~spider who ran up the water spout~~ [a library for working with bits](https://github.com/ThePhD/itsy_bitsy). The goal of this library is to provide a high-quality, publicly available and shippable implementation that will eventually be moved into at first implementation-specific parts of the standard library, and then moved into the standard pending field experience and further design work.

It's pretty barren on the README, docs, etc. It will probably be like that for a good while as I focus strictly on implementation and proving that these range adaptors and container adaptors are implementable, and that they can provide benefits. There will likely be more progress and announcement blog posts later, when the details get finalized, the benchmarks are satisfied, and the performance is justifiable to ship it out.


### bit_view

This is a non-mutable, non-owning view of another view. It takes in a range of `Word`-types (integral types or enumeration) and creates a view of it. That means the following works:

```cpp
// I should really change this to be itsy/
// shouldn't I?
#include <bitsy/bitsy.hpp>

#include <array>
#include <cassert>
#include <ranges>

int main (int, char*[]) {
	constexpr std::size_t b00 = 0x00;
	constexpr std::size_t b01 = 0x01;
	constexpr std::size_t b10 = 0x02;

	// 30 words, with varying
	// bit patterns
	std::array<std::size_t, 30> storage{ 
		b00, b00, b01, b00, b00, b00, 
		b00, b01, b00, b00, b00, b00,
		b01, b00, b00, b00, b00, b01, 
		b00, b00, b00, b00, b01, b00, 
		b00, b00, b00, b01, b00, b10 
	};

	constexpr std::size_t expected_on_bits = 7;

	using R = std::span<std::size_t>;
	bitsy::bit_view<R> view_bits(storage);

	assert(view_bits.size() 
		== bitsy::binary_digits_v<std::size_t> * storage.size());
	assert(!view_bits.none());
	assert(!view_bits.all());
	assert(view_bits.any());

	/* continued below... */
```

Pretty standard stuff, and it all works. There are iterators too, to make one-by-one traversal easier:

```cpp
	/* resumed from above... */

	std::size_t iter_count     = 0;
	std::size_t iter_on_count  = 0;
	std::size_t iter_off_count = 0;
	for (const auto& ref : view_bits) {
		++iter_count;
		if (ref) {
			++iter_on_count;
		} 
		else {
			++iter_off_count;
		}
	}
	assert(iter_count 
		== bitsy::binary_digits_v<std::size_t> * storage.size());
	assert(iter_on_count == expected_on_bits);
	assert(iter_off_count == (view_bits.size() - expected_on_bits));

	return 0;
}
```

Note that `iter_swap` has been changed by the inclusion of Ranges to work better with proxy types too, so it can even be used with a wider variety of `std::` algorithms too!

It's got most of the same methods a `std::bitset` or `vector<bool>` would have. And it has iterators that walk through exactly the number of bits. It still returns the same proxy type like `std::vector<bool>`, but rather than being an exception to the general rules of `std::vector`'s class template, it is completely well-defined as to what it is and how it behaves for all types. This makes it less of a surprise that you cannot take the address of a bit, or similar, because the assumptions come baked into the container for all relevant allowed types!

Very, very nice.

This type is also given `const_bit_iterator`s rather than `bit_iterator`s listed in [P0237](https://wg21.link/p0237). The goal of this was to ensure that there were `const_bit_reference`s also passed out as well, since the `using base_value_type = typename std::iterator_traits<storage_iterator>::value_type` is not const qualified and could result in some interesting shenanigans. One could reasonably just apply the const more directly to the `bit_reference` type but doing `const value_type` or by doing `std::remove_reference_t<typename std::iterator_traits<storage_iterator>::reference>`. The problem with `std::remove_reference_t` on the `::reference` define of `iterator_traits` is simple: "smart references" are a thing. We have special references in `bit_view` and `std::vector<bool>`: it is not inconceivable other containers have the same thing. Therefore, trying to rely on removing the reference-ness of the `reference` type in `iterator_traits` will eventually fail for a type which implements all the proper bit manipulation operations but is a "shim"/"fat"/"proxy"/"smart" reference type.

So, some `const_`-specific iterators. This is already something that is done with iterators implemented for container types today in the standard library, so really nothing too novel is being done here.


### bit_span

This is a mutable, non-owning view of another view. It takes in the range of data it is templated on and produces a `bit_span`. `bit_span` privately inherits from `bit_view` because it has all the same interfaces. It allows all of the same work as `bit_view`, with some additional extra niceties:

```cpp
#include <bitsy/bitsy.hpp>

#include <list>
#include <cassert>
#include <ranges>

int main (int, char*[]) {
	// yep, a linked list of bits!
	std::list<std::size_t> storage{ 
		0x01, 0x02 
	};

	using It = decltype(std::ranges::begin(storage));
	using Senti = decltype(std::ranges::end(storage));
	using R = std::ranges::subrange<It, Senti, std::ranges::subrange_­kind::sized>;

	bitsy::bit_span<R> the_bits(storage);

	assert(the_bits.test(0));
	// assign into the iterator after dereference
	*the_bits.begin() = false;
	assert(!the_bits.test(0));
	the_bits.flip(0);
	assert(the_bits.test(0));

	return 0;
}
```

As shown here, it doesn't matter what the underlying view or range is. So long as it can properly dole out a `value_type` that can be bit-operated with, it works out. This one provides mutable `bit_iterator`s, too, rather than the fully `const_bit_iterator`s of the `bit_view` type, and has functions for flipping and setting bits. We now have a cromulent wrapper around a specific range, and we get all the loveliness that comes with it!


### dynamic_bitset

This is the one with the name I am least happy with, but it is exactly what most people expect it is given existing nomenclature: a set of dynamically expanding bits. This one is not fully implemented in the repository: everything but the properly generic `.insert()` functions are there. This one is, like its predecessors, templated on the input container type: it serves as an adaptor.

Making it an adaptor has several benefits. For example, nothing stops us from writing the following an getting Small Buffer Optimization (SBO) for free on most implementations:

```cpp
#include <bitsy/bitsy.hpp>

#include <string>
#include <cassert>
#include <ranges>

int main (int, char*[]) {
	using my_sbo_bit_set = bitsy::basic_dynamic_bitset<std::string>;

	my_sbo_bit_set sbo_bits("\x53\x21\x01\x00");
	/* off you go! */

	return 0;
}
```

Modulo wasted bits in the null terminator, congratulations: you have a Small Buffer Optimized bag of bits. And you have to do 0 implementation work to get there: just reuse the same container adapter that works for `std::vector`, `std::deque`, etc. etc.

This means that someone could implement a _real_ `sbo_vector` or similar, throw it right in here, and all the same optimizations apply!

Snazzy.



# Phase 2?

Phase 2 is going to be using the newly defined and touched up P0237 iterators to optimize the algorithms, such as `std::rotate`, `std::find`, and more.


### Goal: Algorithms

As proven by [Howard Hinnant 7 years ago](https://howardhinnant.github.io/onvectorbool.html), `std::vector<bool>` can be fast if the standard library specializes its algorithms for the bit iterators. The problem with Howard's work here is that the bit iterators come as an implementation detail to `std::vector<bool>`: nobody can take advantage of the work done in libc++ (and the handful of algorithms for libstdc++) with their own bit containers.

By working to standardize what is in P0237, we can give everyone the ability to have their own bit iterators and similar without loss of performance, and the standard library still only has to optimize its one set of algorithms for one set of bit iteration types. Everyone gets to benefit off of a singular implementation: maximal reusability.

This will be the focus for the next 4 weeks, on top of polishing the rest of `bitsy`.


### Final Goal: Moving into __gnu_cxx

You'll also notice that the internal implementation is littered with `__reserved _Identifiers` and other things. That's because as part of this implementation -- once I am done punting it around in its own repository -- I will be attempting to create a patch to move these utilities into libstdc++. Pending that level of success, I will also attempt to move such utilities into libc++. They do not seem to have a top-level detail namespace (except for `__gnu_cxx`, which they keep around because of old hash map compatibilities in some large code bases, HAH!), so `__gnu_cxx` or `__std_detail` is about as good as any guess right now.

I have already started to get used to the GCC codebase. It is.... well, **massive** is an understatement. The good news is, a large portion of the parts I will be interacting with are pretty clean and self-contained in the `include/bits/` section. All I will have to do is yank out all the defines and swap some more implementation-specific namespaces and headers when moving to libstdc++. I will also port all of my Catch2 tests to the deja-gnu testing suite. I'm going to miss Catch2, but deja-gnu doesn't look so bad to work with! A templated function and I'll probably be right as rain, doing the same test suite over all of the integral types.

Part of this might also be offering a `<bit_range>` header that's non-standard and ships with GCC. It would provide aliases to this stuff in the standard library under `std::bit_view` and friends. Of course, that's a bit weird to do: it might give the impression that its standard when it's not, so I do not think I will be moving for that option anytime soon...!



# Challenges

There are some implementation challenges that need to be worked through over after the algorithms are optimized.


### bit_view<std::reference_wrapper<std::vector<std::size_t>>>

Or otherwise known as: the bane of library developers everywhere. It requires a bunch of really silly `unwrap_unref` calls everywhere and it's fairly exhausting to do, UGH. This is not explicit supported right now, but planned to as part of some Phase 3 if-there-is-time work.


### Higher-Level Constructors

Right now, the abstractions do not provide things like `to_string`, or have specific string-based constructors. This would mess with how the constructors work currently, since what they do right now is just the typical copy/move, as well as forward literally anything else to the underlying storage / range. This gets rid of some of the functionality of `bit_set` and friends: it might be prudent to offer the same kind of functionality, but perhaps add a wrapper or tag so the constructors don't get lost. Alternatively, we make the current pass-through syntax verbose by requiring a `std::in_place` tag to ensure the user explicitly opts into this forwarding-construction, and then provide a set of constructors that implement the from-string conversions and similar.

I'm not exactly enthusiastic about that last suggestion, but it's there and is being considered.


### by-the-bit?

Currently, `bit_view` and `bit_span` just view the underlying `value_type`s. If you want to "bump" the `begin()` or shorten the `end()` by the number of **bits** rather than by a whole `value_type`, this abstraction has nothing for you. However, it is trivial for a user to subclass from `bit_view`, book keep a `difference_type position;` and `difference_type truncate;` member variable, and automatically `begin() + position` and `end() - shift` in their own overridden `begin()` and `end()` calls on their `by_the_bits_bit_view`. (Pending also getting a better name too, hah!)

There is also just going full ranges and doing the initial `bit_view` with a `view::drop` on top of it. There are many ways to achieve what the goals here!


### Bikeshedding

`dynamic_bitset`? `bitset_view`? `bit_view`? `bit_array`? Lots of names, lots of time to get opinion and feedback about what this should be called. It's a lot more general than `dynamic_bitset`, so there's some wiggle room not to go with the same name. Bother me with your suggestions via e-mail, IRC, Twitter, Discord, etc. etc.!



# That's all

There's more work to do, but this begins to lay the foundation for talking about something super serious in the future. It also gives me practice working with container adaptors, which is going to be vitally important for SG16s work on text containers that do not create 20 new string types to be passed around by everyone, but instead work with existing storage to make interfaces better and more idiomatic without 50 overloads for all the `std::*string`s and the `std::*text*` types that might show up.

You can mess with the code in `itsy_bitsy`, but seeing as it's not optimized yet and that names might change it might just be there for eyeballing at the moment rather than anything serious. In the future, the classes themselves will be in either libc++ or libstdc++ (pending a successful GSoC and buy-in from the implementers there) that I will maintain, in preparation for moving these things fully into Standard C++ once all the necessary field experience is gathered.

Until next time! 💚

<sub>Also please don't actually `std::byte`/bite me it was just a pun unless you're an adorable animal and if you are an animal that can read this post can you please make sure not to do it too terribly hard, thank you!</sub>
