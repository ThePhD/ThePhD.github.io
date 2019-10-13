---
layout: post
title: Pack the Bits - Adventures in small_bit_vector
permalink: /small-bit-vector
feature-img: "assets/img/2019-10-13/moving-pack-box-cropped.jpg"
thumbnail: "assets/img/2019-10-13/moving-pack-box-cropped.jpg"
tags: [C++, bit, ⌨️]
excerpt_separator: <!--more-->
---

Strap in for a wild ride of object lifetime, sizing, `constexpr`, and bits!<!--more--> This comes from my need to finish one of the work items I mentioned in my [last article](/seize-the-bits), which was that we needed a proper, dedicated `dynamic_bitset` type to squeeze out every potential bit. The reason for this is that if your container counts in elements -- not bits -- then you need to maintain an extra variable for your container to maintain the current "bit position", which is exactly what the generic `bitsy::bit_sequence<Container>` has [to do](https://github.com/ThePhD/itsy_bitsy/blob/9f4976a0fe1f846a895c5e13f5526a199923b6e1/include/itsy/detail/bit_sequence.hpp#L805).

So, to make sure I knew the landscape well, I implemented it in my library [itsy.bitsy 🕸️](https://github.com/ThePhD/itsy_bitsy). When I started this, I didn't expect it to be easy, but I also didn't expect it to be this hard:

```cpp
===============================================================================
All tests passed (3631334 assertions in 554 test cases)
```

What it took to write all of this tests and get them to pass was really astounding. And even then, I can write even more tests across more a few more types and a few more cases: there's special ways to test exception safety, injecting purposefully failing allocators, and verifying `sizeof()` for various configurations of the container do not exceed e.g. its `std::vector` counterpart's size.

This was my first time building my own contiguous container with:
- Strong exception guarantees
- Allocator support
- Small buffer optimizations (SBO)

I have implemented hash maps, vectors, binary trees and sorted heaps before, but never to the rigor required by something that might go into a Standard Library! It was a great learning experience, even if somewhat painful, and there were a lot of things I got to touch and do.




# Small Buffer Optimization...?

One of the goals of this was to create an optimized container that stashed bits inside of itself if at all possible. This is achieved using what is typically called the "Small Buffer Optimization" or the "Small Storage Optimization" (SBO, SSO). I will use the terms SSO and SBO liberally in this post because typing out the full thing and reading the full thing is a disservice to humanity in general.

The small buffer optimization can essentially be described like internally deploying a union:

```cpp
struct small_bit_unsigned_vector {
private:
	struct small {
		unsigned int buffer[2];
		std::size_t size;
	};

	struct normal {
		unsigned int * first;
		unsigned int * last;
		unsigned int * allocated_last;
	};

	union either_storage {
		small stacked;
		normal heaped;
	};

public:
	// ...
};
```

Here, we essentially guarantee that if the `size()` of the container is less than `3`, we'll store integers in the `small` structure. Otherwise, we heap-allocate and use the usual three pointer layout! Simple enough.



## ... Seriously? Less than 3 integers?

It's just an example, but it does highlight a bit of the problem: typical SBO in `std::string` in, say, MSVC or similar can get a good amount of code units since the string is templated on `char`. The space savings go down with `wchar_t`, `char16_t`, and `char32_t`: trying to fit things in the same storage as 3 pointers does not give a whole lot of extra space, even if the pointers get larger in 64-bit builds.

But! Maybe the user is okay with having a size greater than 3 `int`s in the small buffer. Or maybe they -- for some reason -- want just 1. The problem with `std::basic_string` implementations today is that the SBO is an implementation detail that occasionally leaks out into the interface due to moves invalidating iterators (unlike `std::vector<T>`, which does not do this). Worse, it is impossible to know just how well-packed a standard library implementation made their structure. Is it 7 `char`s? 8? Does it change between 64-bit and 32-bit? Between `char` and `unsigned char`...? "Unspecified" and "Implementation defined" are always the enemy of cross-platform code. So, we come to the simple solution:



## User Defined SBO!

If we just let the user pass in their own template parameter, they get to control how much they want for the size of the inline container. Thusly:

```cpp
template <std::size_t InlineN>
struct small_bit_unsigned_vector {
private:
	struct small {
		unsigned int buffer[InlineN];
		std::size_t size;
	};
	// ...
};
```

Now, the user knows how much inline space they pay for any given `small_int_vector<N>`, modulo the `normal`, heap-based representation of 3 pointers! I defined a `bitsy::default_buffer_size_v<T, ...>` in my implementation to use as the default value that attempts to keep the SBO size within the triple pointer storage size, but that's just a default! So far, everything looks good.

... Except, if you were at CppCon 2019 and saw CJ Johnson's talk _[How to Hold a T](https://www.youtube.com/watch?v=WX8FsrUbLjc)_ you would probably frown when seeing the above, unnerved by how _wrong_ the definition above is for a smaller buffer type. See how the internal inline buffer is just declared as a plain old C-style array? That's actually a problem. If you were to make this class above even more generic and insert a class in there, you'll notice something interesting:

```cpp
template <typename T, std::size_t InlineN>
struct small_bit_vector {
private:
	struct small {
		T buffer[InlineN];
		std::size_t size;
	};
	// rest of the definition...
};

#include <iostream>

struct non_trivial_unsigned {

	non_trivial () {
		std::cout << "AAAAH" << std::endl;
	}

	unsigned data;
};

int main () {
	small_vector<non_trivial, 5> sbo_vec;
	return 0;
}
```

This program's output:

```cpp
AAAAH
AAAAH
AAAAH
AAAAH
AAAAH
```

... Oopsie.




# You Will Initialize and You Will Like It

C arrays (and `std::array`) in C++ default initialize their elements when you don't provide an initializer or construct the entire array at once. For trivial types, it's essentially "garbage initialization" (and generally serves as one big language-wide security hole that people are [constantly trying to get around in their compilers](http://lists.llvm.org/pipermail/cfe-dev/2018-November/060172.html)). But for non trivial types, default initialization is almost always the same as having and calling a default constructor.

This might not seem like a big problem, but it turns out to be big later. If you were to:

- insert a `tracking_allocator` that counted the number of constructions and destructions of `T` from the container;
- and, added counters in the class to count the number of constructions and destructions,

then the counts would mismatch. Not to mention that in the above, types that do not have default constructors and do not have them generated by the compiler would utterly fail compilation. There are 2 ways to fix this: the typical (old school) runtime way, and the new, shiny way.



## Run-time Style

In this case (and in some ye olde implementations and blog posts), you would use a `unsigned char buffer[InlineN * sizeof(T)];`. Or, newer, a `std::byte` array or a `std::aligned_storage` type. Then you would
- `reinterpret_cast` to `T*`;
- keep track of the size;
- and, initialize one element of `T` at a time (or destruct one element of `T` at a time).

Unfortunately, this has its own problems: casting an array of `char` or `unsigned char` or `std::byte` or whatever's in `std::aligned_storage` is not legal `constexpr`-time C++! And with `constexpr` `std::vector` and `std::string` coming to C++20, we need to prepare for a future where this class will need to work at `constexpr` time. So...



## `constexpr`-friendly Style

The way to do this is to set up a "lifetime cage" or a "delay-initialized `T`". The best way to do this is to realize that the compiler cannot -- and will not -- default initialize a union: 

```cpp
struct non_trivial {
  
  non_trivial() = delete;
  constexpr non_trivial(int d) : data(d) {}
  
  int data;
};

union cage {
	
	constexpr cage () : dummy() {}
	constexpr cage (int d) : meow(d) {}
	
	char dummy;
	non_trivial meow;
};

int main()
{
	static constexpr cage locked_meows[500];
	return 0;
}
```

There we go, now we have a way to make `std::array` or C arrays of a fixed size but not call the default constructor! So we use that in our definition above, and now we can construct each array element one at a time, and destruct them one at a time as well! This fine-grained control means we get to meet construct/destruct counts, which is something I spent a [lot of test resources ensuring correctness](https://github.com/ThePhD/itsy_bitsy/blob/9f4976a0fe1f846a895c5e13f5526a199923b6e1/tests/run_time/source/small_bit_vector.allocator.cpp#L27).

Perhaps the only problem with this technique is that you cannot treat a sequence of `cage` as a sequence of `non_trivial` at compile-time. That is, this is -- in colloquial terms -- HELLA wrong and illegal:

```cpp
// ...

#include <cassert>

int main()
{
	static constexpr cage locked_meows[500]{1, 2};
	// okay, developer-san! 😊
	static constexpr const non_trivial* first = &locked_meows[0].meow;
	// ... aheheh, everything's still fine...! 😅
	static constexpr const non_trivial* second = first + 1;
	// W-Wait, d-don't please I beg of you! 😰
	static constexpr int two 
		// DON'T DO IT DEVELOPER-SAN !! 😱
		= second->data; // H͌Ǒ͕͎̀W̗͔ ̪̀D̈́A̳ͅR͔̰̃͂E̾ͮͪ Y̜̱͍ͨ̌̉Ő̟͍̂Ụ̂ ͍AṄ͙̙͑GE̦̗ͪ͐R̆ ̫̰̼͔̦͍̜̞̙̠͙̯̺ͬ̃ͣ̅̊̍ͦ̇ͬ̃̋̊̓
	ḁ̯̲̣͉̀̄ͭ̓͐sͩͤ̇̐͆s̼̼er̫͊t(̪̻̍ͧt̺̖̝̟̮ͫ̎̔̍ͥw͖̆o̅̍ͣͩͧͯ ̤͖̪̤̤̼̍̏͌ͥ̾̚=̙͓͍͇ͦ͊͌͆=̙̯̪
```

And developer-san was never heard from again.

Silliness aside, this -- unfortunately -- errors. At runtime it will work (because the memory is properly laid out, so why wouldn't it?). But `constexpr` doesn't care about your silly notions like memory layout and "as good as": it has hard limits and guarantees to make based on the virtual parameterized non-deterministic abstract machine. We have to write special iterators just to walk this storage, dereference into `data`, and then give back an `int&` to the user. The price to pay for `constexpr` stuff ...?

Ignoring that (little?) problem, we now have ways to have our strong SBO container after these techniques! And yet, the nagging feeling remains...




# Is That the Best We Can Do?

To answer the question early, no. There are many different ways to implement an SBO besides what was outlined above. I tried 2 separate implementations with different ways of discriminating whether or not we are "within SBO" or "on the heap". They have different consequences and tradeoffs, and really only one of them is suitable for Standard Library Containers and the guarantees they tend to make. Still, we will discuss all the alternatives, and what problems they may or may not have!



## Strategy 0: Consistent Size, Size determines use of SBO

This technique keeps the size in some fixed area of memory and uses it to tell the overall container whether it is within the bounds of the small buffer or not, which makes fairly direct sense:

```cpp
template <typename T, std::size_t InlineN>
struct small_bit_vector {
private:
	struct small {
		cage<T> buffer[InlineN];
	};

	struct normal {
		T* first;
		T* capacity;
	};

	std::size_t bit_size;
	union {
		small stacked;
		normal heaped;
	};

public:
	// constructors and member functions ...
};
```

This technique works pretty well, for almost the entirety of `small_bit_vector`. Astonishingly so: it also ensures that you can use at least 2 spaces worth of `T*` for a packed representation before you start getting competing with the typical `std::vector<T>` representation.

The only problem with this design is a call to `.erase()`.

Imagine, for a moment, you have this container `small_bit_vector<std::uint64_t, 2>` on a 64-bit machine. You have 129 bits in this container. A call to `this->is_small_buffer_optimized();` returns `false`. It has already "spilled" past the edge of the SBO, and it exists solely on the heap now. You then call `.pop_back();` and now have 128 bits. Internally, this container calls `this->is_small_buffer_optimized();`...

And it now reports `true`.

In order to use the `bit_size` as a discriminant, you would need to eagerly shrink from heap storage to small storage whenever you destroy enough bits. This is not what most people expect from their containers, because it means you _invalidate all existing iterators_ 😱!

If a container was okay with this though, Strategy 0 gives the most SBO space possible.



## Strategy 1: Pointer to First

This technique keeps a `T*` in some fixed area of memory and uses it to tell the overall container whether it is within the bounds of the small buffer or not:

```cpp
template <typename T, std::size_t InlineN>
struct small_bit_vector {
private:
	struct small {
		std::size_t bit_size;
		cage<T> buffer[InlineN];
	};

	struct normal {
		std::size_t bit_size;
		T* capacity;
	};

	T* first;
	union {
		small stacked;
		normal heaped;
	};

public:
	// constructors and member functions ...
};
```

This technique is much less straightforward. Every time the internals transition from `small` to `normal` storage, the `first` pointer needs to be adjusted. The test for figuring out if the type is on the heap involves getting pointers to the class itself, casting, and then comparing values. At worst, you invoke non-`constexpr` sadness by having to cast `this` and `this + 1` into `T*` to do the "am I in small storage" check. At best, you access the potentially non-active member of the union in `stacked.buffer[0]` and `stacked.buffer[InlineN]` to check if it falls within those two addresses. Neither are very friendly to `constexpr` and a rigorous compiler that tracks the active member of a union will probably give you a big middle finger if it detects what is being done.

Perhaps you can cast to `void*` then cheat with `std::greater_equal<void*>{}(void_first_pointer, low_void_pointer)` and `std::less<void*>{}(void_first_pointer, high_void_pointer)`. Except, if you have an allocator, the `pointer` and `void_pointer` types on it might not be amenable to this! Also, the C++ Standard officially states that pointers of 2 unrelated types are not required to have the same size or representation, so doing this is already invoking implementation defined behavior. (Windows and POSIX both define data and function pointers to be of the same size and as far as I understand do not employ special data pointers for individual sizes, so that's something of a blessing.)

Just thinking about this makes my head spin, and worries me in the case of `constexpr` `std::string`. Will the memory layouts and choices be different from big and small strings at compile time versus runtime? In a C++23 world -- if we don't solve these problems -- does that mean the memory usage for a 4 `char` `constexpr` `std::string` is different than a regular runtime `.size() == 4` `std::string`? What happens when we allow allocations to escape `constexpr` and we lower the string representation into program memory in some observable fashion? Can we guarantee our containers will continue to work?

... Aaaaah! 😵

These things worry me, perhaps more than it should. I'm sure some more clever applications inside of `constexpr` can make this technique work out better, though. I'm going to need to read over the Standard a lot more carefully to make sure I'm not digging myself a hole here during the `constexpr` container implementation wave in the next year or so.

It also makes the size part of the special storage, robbing potential bits from a highly compact representation that wants to keep the same size as a `std::vector<T>`. It solves Strategy 0's problem of having to shrink to the `small` storage again when `.erase()` is called, because the bit size is no longer part of the discriminant that tells the class in which object of the `union` to look. This means it grows and it stays on the heap until an explicit request to `.shrink_to_fit()`. Iterators remain valid sans the case of `.insert()`/`.push_back()` and friends. But, that's par for the course with standard library containers and mostly expected by comfortable working programmers these days.



## Strategy 2: Pointer to First, Pack the Size

This strategy comes solely from recognizing that having a `std::size_t bit_size` inside of the `small` storage is wasteful. Do we really need all 32 or 64 bits of the `size_t` to represent `InlineN`? Chances are the answer to that question is "nope!".

Because this is a bit container, we recognize that we can steal bits off of `T` to store the size, while allowing a few extra bits that are not part of the size. In particular, we only need `log2(InlineN)` bits to represent the SBO size. So, if we have:

- `log2(InlineN) < bitsy::binary_digits_v<T>`;
- and, `std::is_integral_v<T>`

then storing the size in the most significant `log2(InlineN)` bits ("leftmost" bits in C++ bit operation terms) is totally possible! The implementation in `itsy.bitsy`, unfortunately, doesn't do this at the moment. Some of the algorithms check, use, and manipulate the size in the middle of adding and removing elements. This creates some places where we overwrite bits that were not meant to be overridden. This also gets in the way of performing copy operations of whole `T`s at a time, rather than individual bits, since the last `T` in any given hyper-optimized storage has size information not amenable to the copy operation (meaning we have to be careful and cherry-pick bits in some cases).

Still, it's completely possible and perhaps later I will go back in and [flip the switch](https://github.com/ThePhD/itsy_bitsy/blob/9f4976a0fe1f846a895c5e13f5526a199923b6e1/include/itsy/detail/small_bit_vector.hpp#L129) and guide myself through the process of vetting the entire implementation. I believe that MSVC's `std::string` also does this to get additional savings out of the string, leaving a `char` for count in its arrays and a null terminator too, leading to about ~7 `char`s worth of SBO space (for 32-bit, more for 64-bit if I remember correctly).




# And There's More!

So, sooo much more! Allocators were mentioned here (but that deserves its own article). There are optimizations that still need to be done in `bitsy::small_bit_vector`/`bitsy::dynamic_bitset`, too. Bulk insertions and bulk erasure need special love! But, well. That will have to wait...

for next time. 💚

<sup><sup>P.S. allocators are the devil and default initialization should be allowable with a special tag to construct calls and a special noinit syntax in the language don't @ me.</sup></sup>