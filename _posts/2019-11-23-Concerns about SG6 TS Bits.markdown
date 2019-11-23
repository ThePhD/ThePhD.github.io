---
layout: post
title: Early Feedback on SG6 bits
permalink: /concerns-sg6-numerics-ts
feature-img: "assets/img/pexels/drill-bits.jpg"
thumbnail: "assets/img/pexels/drill-bits.jpg"
tags: [numerics, ts, sg6, C++, bits, 📜️]
excerpt_separator: <!--more-->
---

SG6 Numerics has put out [a draft Technical Specification](https://wg21.link/p1889) to get the ball rolling on better facilities for handling all kinds of numbers in C++. Naturally, I laser-focused on the parts providing access to bits and bit containers. It is nice, but I've got some<!--more-->

_opinions_.




# Repeating Old Mistakes: `std::bits`

The Numerics WIP paper [P1889](https://wg21.link/p1889) has a type called `bits`, which purports itself to be an "object that represents an unbounded set of bits" (§9.2). This is similar to `boost::dynamic_bitset` and many other versions that have come before it.

Reading the specification makes it entirely easy to see this is just a slightly modified version of `std::bitset<N>`. That means it makes the same mistakes as `std::bitset<N>` too. Here's the quick dirty laundry list:

- The constructors are a mess.
	- It has a constructor which takes a `std::basic_string`, giving a hard dependency on not only the `<string>` header but also forcing users of any container that isn't a string to fall into the trap of accidentally copying their data into that string.
	- Did you have a data type that cannot be broken down into a character type pointer and its size? Well, your type is not welcome in any of the constructors.
	- There are no constructors that take `Iterator, Sentinel` pairs like all of the other containers.
	- The constructors and member functions mix serialization and conversion ("print zeroes and ones") into the data structure.
	- `bits(initializer_list<uint32_least_t>)` is a slideware constructor. It does not help me with my data that may be 64 bits or larger for each element. This is likely thrown in as a bone for `bits{ 0xFF,0x560033, ... };` usages, but it's an extremely awkward way to do it.
- There are no ways to control the underlying representation of what is stored.
	- Is it `size_t`? `uint32_t`? `__uint128_t`? The standard library just picks something, and to hell with you or your thoughts on the subject or what's appropriate for your work load.
	- Small buffer optimization is not a first class citizen. The push for `small_vector<T, N>` and other data structures has made it clear that being able to control an implementation's small buffer size is critical for high performance applications, and just vigorously shrugging our shoulders and saying "Quality of Implementation issue" is very much _not okay_.
- There are no iterators.
	- "You didn't need any of that algorithm stuff, let alone those Ranges thingies, right?"
- There is no option for small buffer optimization.

Given the size of the paper (and the many other things which they get right and focus on), I assume these oversights were mostly due to just following existing practice of `std::bitset<N>`. I am certainly glad that they published the work and are moving Numerics forward, but it will be over my cold, lifeless body that they repeat the mistakes for `std::bitset<N>` in a C++23-era Numerics type.




# Alright, It's Bad. Now What?

I'm not a fan of complaining about stuff for the sake of complaining: providing solutions is necessary. While step 0 is to get noisy about it -- never let anyone tell you not to be noisy about broken things -- step 1 is to actually propose some fixes. Yelling from an ivory tower about a purely academic or theoretical design is not worth much, why is why

[everything illustrated here will have implementation experience](https://github.com/ThePhD/itsy_bitsy).

Of course, it is not just me: Alisdair Meredith and Vincent Reverdy [already wrote papers to fix much of what is related to bits in the Standard Library](/seize-bits-production-gsoc-2019), some of those papers dating back 13 years. While we have just blissfully twiddled our thumbs about the situation for binary workloads in the Standard Library, if we are going to bundle `std::bits` as a part of Numerics we should do it right.



## Fix 0: Drop the String Constructors

Drop the `std::string` constructors and stop strongly coupling the containers and a potential form of its serialization. Whether or not compilation times get worse because of the header is a secondary concern: modules or no, this kind of constructor is the pinnacle of mixing concerns and is not something that should show up in an API designed for today's use.

Even a `std::string_view` constructor is of supremely bad form. These should be separate free functions and have no business on the interface of the class: that they exist is a symptom of not having a proper way to iterate over bits in the Standard Library, and as soon as Fix 0 is deployed the pressure of providing these serializing constructors decreases dramatically.

Serialization can be provided by a suite of functions similar to `to_string`. They can take this form:

```cpp
template <typename Iterator, typename Sentinel>
std::bits to_bits(Iterator first, Sentinel last);

template <typename Iterator, typename Sentinel>
std::bits to_bits(Iterator first, Sentinel last,
	iter_value_t<Iterator> one_value,
);

template <typename Iterator, typename Sentinel>
std::bits to_bits(Iterator first, Sentinel last,
	iter_value_t<Iterator> one_value,
	iter_value_t<Iterator> zero_value
);
```

Note the templated free functions have no hard dependency on `basic_string_view` or `basic_string` is present, and also allows for a plethora of implementation strategies and better ways of defining what `zero` and `one` should be for the computed `value_type`s of the iterators. Furthermore, ranges we can shrink the argument count to just 1 in the simple case:

```cpp
template <typename Range>
std::bits to_bits(Range firstlast);
```

This is a far better design that reduces coupling.



## Fix 0a: Better Constructors

Similarly, the absolute _lack_ of any of the modern container constructors on the entire class is extremely worrying. `itsy.bitsy` contains an example implementation with the full suite of `count, value`, `iterator, sentinel`, and `initializer_list<value_type>` constructors, as well as an extended set of constructors for all of these but with "bulk-copy the underlying values". This provides maximum flexibility for initialization **as well as** performance for people who know the most efficient way to initialize their data.

It follows _better_ existing practice already in the Standard Library; let's not ignore a good thing.



## Fix 1: Proper Iterators

In `itsy.bitsy` there are bit iterators, modeled after a paper already sent to the Standards Committee and on its 10th+ revision. There's no reason to not put those iterators on `std::bits` and `std::bitset<N>`. Not being able to use standard library algorithms with standard-like containers is something of an sad joke. itsy.bitsy also proves that by providing these iterators, you can optimize the standard library algorithms appropriately **and** make it so other people can use `std::bit_iterator`s properly and get the same performance benefits.

This is fundamental to having `std::bits` be generally accessible to all levels of programmers. It will unlock its use in many different contexts, far beyond the one-off uses of `std::bitset<N>`; creating types which are incompatible with the goals and creeds of the Standard Library at large is destined to make `std::bits` the equivalent of annoying vaporware, and far less powerful than `std::vector<bool>`.



## Fix 2: Turn it into a templated container type.

The siren's song of taking the template type off a container to save on compilation time is a lovely one to listen to.

It's less lovely when during important project and the types you selected that underpin the data structure do not match my project's needs.

At no point in my existence is "I reimplemented `std::bits` but with a different underlying type and allocator support" a good use of anyone's Engineering Time™. Even `boost::dynamic_bitset<Block>` takes a type which serves as the underlying value; writing a container which is strictly inferior to current standard containers and existing practice is not something to be standardized. `std::bits` should be templated on a type `T` to perform the underlying bit math on. This also ties into Fix 1: now developers can have constructors that always construct unambiguously from an `iterator, sentinel` pair that traverses `bool`eans, or developers can create an "extended" constructor [like is present in `itsy.bitsy`](https://github.com/ThePhD/itsy_bitsy#bit-sequence) to directly fill in `T` objects that represent the bits.



## Fix 3: Specify a small buffer optimization up front.

Clearly, the type is meant to be a vector-like type. It has `operator[]`, `size()`, `reserve()`, `capacity()`, and `shrink_to_fit()`. We already have the problem in the standard library right now that `std::basic_string` is our only de-facto SSO container, and some people have already noticed that they can cheat and do `std::basic_string<double>` and get a mostly-working Small Buffer Optimized container. (That is not standards-mandated to work, but it sure as hell compiles on a lot of platforms).

Don't start with `std::bits<T, Allocator>`. Start with `std::small_bits<T, N, Allocator>`. Note that `std::bits<T, Allocator>`'s implementation becomes trivial after successfully implementing `std::small_bits`:

```cpp
template <typename T, typename Allocator> 
class bits : private ::std::small_bits<T, 0, Allocator> {
	/* forward constructors/members here */ 
};
```

You can kill 2 birds with 1 stone, and provide a container which has strong vector-like iterator guarantees if `N == 0`: otherwise, you get the same iterator guarantees as `std::basic_string`.




# Is That It?

Not exactly. For example, nowhere is there a type to view a set of bits that already exist with something like `itsy.bitsy`'s `bit_view` type, making it hard to take pre-existing storage and "view" it as a sequence of bits to operate on. But, given that the paper only had `std::bits`, it felt appropriate to only tackle the problem of an unbounded bit container.

The old design got us out the front door, but it has been a very long time. While back then we did not know any better, we are better than our old mistakes. If we are going to find better replacements for `std::vector<bool>`, they better _actually_ be... well, better. We have enough exemplary work in the wild already showing that it is possible.

Let's do it right, okay? 💚
