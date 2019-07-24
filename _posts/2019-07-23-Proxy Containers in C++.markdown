---
layout: post
title: Container Adaptors - A deep inspection of a generic dynamic_bitset
permalink: /container-adaptors-gsoc-2019
feature-img: "assets/img/2019-07-23/adapter.jpg"
thumbnail: "assets/img/2019-07-23/adapter.jpg"
tags: [C++, GSoC, bit, ⌨️, 📜]
excerpt_separator: <!--more-->
---

Adapter? Adaptor...? Either way, [last time](/proxies-references-gsoc-2019), we focused on the problems of proxy iterators in many of its forms. This post is going to focus specifically on what we can learn<!--more--> from that, and how it directly impacts the iterators and containers of the types I am developing for the Summer of Code 2019.


# But first, a Status Report

Today's learning comes from trying to quickly finish up the end of my Summer of Code for bit containers and move onto algorithms, only for performance and memory footprint to come smack me squarely in the nose. This slightly derails the original plan to focus on the algorithmic portion of the optimizations (e.g.: optimize a handful of `std::` algorithms and follow in Howard Hinnant's footsteps), but is no less important or consequential for not just bit handling in the standard library, but "Proxy Containers" in general.

Thankfully, I was prepared for this: the last segment of the Summer of Code was essentially left up to open-ended exploration of a few topics around relocation and performance of trivial types like bits that are essentially relocatable. It also was meant to serve as "cushion time", in case Standardization work and other things clobbered my mid-summer. Seeing as it has, the third phase thankfully has enough overflow time to focus on committing more work and working with whatever things I happened to discover over the time of my Summer of Code work!

I also got to get used to the GCC patch process by submitting a relatively inconsequential patch implementing one of my own accepted papers for C++20: p1301 - `[[nodiscard("should have a reason")]]`. The patch is [here](https://gcc.gnu.org/ml/gcc-patches/2019-07/msg01342.html) and after I clean it up it might land in GCC! This gives me some of the experience necessary to contribute to GCC -- and, in turn, to libstdc++ -- near the end of this Summer of Code.

So, given that we have some breathing room and aren't going to be crazy rushed to complete the deadline, let us focus on some of the problems encountered since the [last update](/std-byte-me-gsoc-2019)...


# Drilling down: basic_dynamic_bitset ?

The issues from the last post are about proxy iterators, and everything (except for the case of unequal iterators) is generally solvable. It only gets into more fun, however, when _container adaptors_ come into play! In fact, the "problem" started back in C++11 for generic proxy containers, when an objectively correct but damning little change was done to all the containers and their `insert` and `erase` functions...


### `const`ipation

If you look on [cppreference for std::vector<T, Alloc>::insert](https://en.cppreference.com/w/cpp/container/vector/insert), you'll notice C++11 made a tiny, tiny little change to how `insert` and friends are specified. `insert(iterator, const value_type&)` became `insert(const_iterator, const value_type&)`. When writing `basic_dynamic_bitset`, my `insert`s and `erase`s originally took `iterator`. But, well, that's not very Standard of me! We'll just change it to `const_iterator` to match the standard, and then--

> error: attempt to modify `const std::size_t&` ...
> 
> error: no conversion from `std::vector<std::size_t>::iterator` (AKA `__gnu_cxx::normal_iterator<const unsigned long long*>`) to `std::vector<std::size_t>::const_iterator` (AKA `__gnu_cxx::normal_iterator<unsigned long long*>`)

... Uh oh.

As you might have begun guessing from this error message (incredibly slimmed down with all the template spew thrown out the window), there is no way to convert from `const_iterator` to an `iterator`. This makes sense: that would violate `const` rules! but, why was _I_ getting these errors in my writing of `basic_dynamic_bitset`? The problem arose from, once again, dealing with the fact that I was presenting an interface that worked on bits, when I was really trying to modify whole words...

And this leads to very big problems.


### Untouchable Iterators

When I first wrote `basic_dynamic_bitset` to take `iterator`, I had inadvertently given myself a huge shortcut: I could immediately pry open my `dynamic_bitset`'s `iterator` with a simple call to `.base()`. Then, I would simply shove the new bit in, and then right-shift all the integer's bits over one (preserving whichever bit "fell off" the right side and propagating the change over to the next bit, all the way to the end of the list). With a `const_iterator`, I would dereference it and get a `const IntegralType&`, which I cannot use to modify bits. Trying to do `iterator __it = __some_const_it;` was also verboten, resulting in the second error up above.

The fix was not pretty.

For any `const_iterator` I received, I needed to essentially "walk" myself with a normal, non-const `iterator` to the position where the insertion was to be done:

```cpp
iterator insert (const_iterator __pos, bool __val) {
	using ::std::begin;
	using ::std::next;
	// ...
	// UGH ... disgusting!
	__base_iterator __storage_first = begin(this->__storage);
	auto __diff = ::std::distance(__base_const_iterator(__storage_first), __pos.base());
	__storage_first = next(__storage_first, __diff);
	// rest of the work on __insertion_position here...
	// ...
}
```

For a typical random-access iterator, this imposes very little cost: jumping to the right pointer value is cheap. But for a non-random access iterator (e.g., for `std::list`), this is quite terrible. Given that we may wrap far more complex data structures (many being custom tree and lookup structures), re-walking the tree/list/etc. is not really acceptable.

What do we do?


#### Option 1: Cheat

Just go back to using `iterator`. Sure, nobody will be able to use the `insert` algorithm with a `const_iterator` (which will come from a lot of places), but it does eliminate the concern entirely. This is the lazy way to do it and likely will not go over well if put in a real C++ proposal to have these types standardized with LEWG.


#### Option 2: Conform to the `std::` interface

Stick with `const_iterator`. This means we need to stay with the poor performance for `std::list` and friends that are not random-access, paying a 2N iteration cost (once to get the difference, once to get move the iterator into place). We can reduce this penalty by using `if constexpr` and some `decltype()` + `is_detected` fun to check if the iterator can have its difference size computed by just calling subtraction. Or, use some of the new `std::ranges` concept machinery to check `random_access_iterator`. If it matches the concept, we just defer to `std::next` and `std::distance`. Otherwise, we don't use `std::distance` and instead purposefully increment up all the way to where we need to be:

```cpp
iterator insert (const_iterator __pos, bool __val) {
	using ::std::begin;
	using ::std::next;

	// ...
	__base_iterator __storage_first = begin(this->__storage);
	if constexpr(::std::random_access_iterator<__base_iterator>) {
		auto __diff = ::std::distance(
			__base_const_iterator(__storage_first), 
			__pos.base());
		__storage_first = next(::std::move(__storage_first), __diff);		
	}
	else {
		__base_const_iterator __comparison_it(__storage_first);
		auto __limit = __pos.base();
		for (; __comparison_it != __limit; ++__comparison_it) {
			++__storage_first;
		}
	}

	// rest of the work on __storage_first here...
	// ...
}
```

This takes the cost down to N rather than 2N for getting where we need to. We can then do all the usual bit manipulations on the non-const iterator.



# ... That's it?

That's it. That's all we can do.

... Sort of.

The underlying problem here is that `basic_dynamic_bitset` is a generic wrapper over a container. When someone hands a `std::vector<bool>::const_iterator` to `std::vector<bool>` and it needs to do much of the same operations, it's okay because `std::vector<bool>` knows how to take its own `const_iterator` and turn it into a regular `iterator` with which to do manipulations in a safe manner. There is no way I can know how to take Microsoft's debug-wrapped `std::deque<T>::const_iterator` and turn it into a normal `iterator`. Nor can I take a `__gnu_cxx::normal_iterator<const unsigned long long*>` and turn it into a `__gnu_cxx::normal_iterator<unsigned long long*>`.

... But maybe the implementations can.


### Customization point: `iter_as_mutable`

If the only problem is that I need to obtain a mutable iterator from `__pos.base()`, then I should give the librarian who made the iterator a chance to directly convert that iterator to a mutable version of itself using some implementation-specific knowledge. `iter_as_mutable` is what I am going to spend the next two weeks implementing and polishing for various libstdc++-specific iterators to allow me to not have to do the `::std::distance` or the manual incrementation dance to avoid the pains of non-random-access-iterators. If an implementation provides a `iter_as_mutable` ADL customization point which takes their `const_iterator` and turns it into an `iterator`, I can avoid all of the song and dance and just directly retrieve the necessary iterator before modifying the bits.

I can write another `is_detected_v` test to check if the `iter_as_mutable` call is legal:

```cpp
template <typename T>
using is_iter_as_mutable = decltype(iter_as_mutable(std::declval<T&&>()));
```

And use it like so:

```cpp
if constexpr (is_detected_v<is_iter_as_mutable, __base_const_iterator>) {
	__base_iterator __mutable_pos = iter_as_mutable(std::move(__pos).base());
	// yay!
}
```

There. Now we have a good escape hatch which we can take advantage of in the libstdc++ implementation by defining conversions for things like `__gnu_cxx::normal_iterator<const unsigned long long*>` and pointer types like an plain ol' unwrapped `const unsigned long long*`. We can also define it for `std::list`'s iterators, and more! So, at the very least we have a default general implementation that does a best-effort, and otherwise we have an extension point that allows for maximum speed.

Very cool!


# Other optimization problems

The last problem we have to face is the choice of container inside of `basic_dynamic_bitset<Container>`. In the general sense, a `basic_dynamic_bitset<Container>` must maintain not only the underlying `Container`, but a separate `bit_position` to keep track of how many bits of the last `typename Container::value_type` it has used so far. This is wasteful if the underlying container already maintains its position in terms of bits, or if the container _could_ maintain its position in bits but doesn't for various obvious reasons (e.g., a `std::vector<std::byte>` implemented using two `std::byte*` would compute size by subtracting pointers, and that makes it impossible to efficiently keep a sub-byte size).

There are a few ways to go about solving this kind of problem.


### Specialize for all interesting templates?

One could conceivably spend all their time creating specializations that conform to the interface but optimize based on the container that `basic_dynamic_bitset` wraps. For example, `basic_dynamic_bitset<std::vector<IntegralType>>` could be optimized to essentially relocate the guts of `std::vector` into `basic_dynamic_bitset` and then keep a `pointer` + `bit_size`. The problem with that is similar to the current state of `std::vector<bool>` and would subvert expectations in subtle and rather annoying ways. In particular:

- calling `std::vector<std::size_t>& = my_dyn_bitset.base();` would not yield a "proper" `std::vector` reference, causing problems for code which wants to access the underlying container and pass it off to other routines;
- reimplementing `std::vector` or `std::list` just with a different storage tracking strategy is not a good use of implementer time;
- and, it leaves out just giving meaningful vocabulary to "a container that handles bits".

Given those reasons, it likely would be better to give a type that explicitly manages things at the `bit` level, without tying it to the `basic_dynamic_bitset` adaptor. So...


### Small Bit Buffer?

To save on that last bit of space, one could conceivably turn to a `small_bit_buffer<WordType>`. The reason we would want such a type is so that a call to `my_dyn_bitset.base();` always yields a proper reference to the `small_bit_buffer` type without going through any implementation shenanigans. It also communicates to the `basic_dynamic_bitset` adaptor that it does not need to maintain a `bit_position` or a `bit_size` variable, since the `small_bit_buffer` type will handle that. Finally, we can do interesting things with `small_bit_buffer`, like address the fact that many memory critical applications will take advantage of the fact that it can cram all of its keys and storage in, say, 256 bits, which we can have in a type dedicated to the Small Buffer Optimization.

A type like `template <typename WordType, size_t SBOSize> small_bit_buffer` could be vastly helpful. We can specialize safely knowing that `small_bit_buffer` will keep track of the by-the-bit size, and `basic_dynamic_bitset` will take care of making a nice interface for precisely the bit part. Still, this isn't much better than the last solution: despite us creating a specific container for the purpose, we have failed to let others slot in their own bit-counted containers and trees here. There would likely need to be some way of opting in to such a space optimization... but what to call it? Do we ask for a `bit_size()` member function and check for that? Perhaps a `is_bit_sized_v<T>` trait to specialize for their type, and have `.size()` return the bit size?

Perhaps yet still, maybe there needs to be a `bit_traits<C>` type, similar to `allocator_traits`, where one can specialize certain actions like retrieving or updating the size. There is a lot to explore in this space. One way that `small_bit_buffer` can participate and keep the size optimizations is to left-shift its size value over by `log2(binary_digits_v<WordType>)`: this would leave enough space to use the bottom `log2(binary_digits_v<WordType>)` bits for the bit position, while perfectly retaining both the size in `Word`s plus the size in bits!


# All the potential optimizations?

Other than using template parameters to indicate a specific bit range on something like `bit_view`, I have not found any additional optimization opportunities. So, having surveyed the space as fully as possible, I think I am going to go with the `small_bit_buffer` optimizations for both speed, SBO and size. I will also be implementing the `iter_as_mutable` internally and trying to see how well it works for things like `__gnu_cxx::normal_iterator<Pointer>` and such types!

I'll catch you later. 💚
