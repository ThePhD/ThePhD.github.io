---
layout: post
title: Reconstructible Ranges and Good API Design
permalink: /reconstructible-ranges
feature-img: "assets/img/2019-07-25/architecture-blue-build.jpg"
thumbnail: "assets/img/2019-07-25/architecture-blue-build.jpg"
tags: [C++, ranges, concepts, ⌨️, 📜]
excerpt_separator: <!--more-->
---

Otherwise known as "oooh, fancy!". In all reality, it's not actually a complicated or even new concept, just<!--more--> a propagation of an old idea. All containers can be created from an `iterator, iterator` pair: this allows someone to take any `Container` type and know they can create a new one by doing `Container(first_iterator, last_iterator)`. Thusly, a simple idea:

_most ranges should be reconstructible from their own iterator/sentinel pair._

And so, from this, a concept emerges...


# Reconstructible Range

`ReconstructibleRa`-- er. `reconstructible_range` is a concept that tells someone whether or not their view type can be pulled apart into its iterators, and then put back together into itself.

I first ran into it when I began to develop the interfaces in [P1629 - Standard Text Encoding](/_vendor/future_cxx/papers/d1629.html). There, I was creating range-based algorithms wherein someone would hand me a `std::u16string_view` or similar for an input. After performing some work with the iterators, I wanted to return the same `std::u16string_view` to the user, just with the view "changed". I was doing this in a generic algorithm, so I simply took the type name and used the same convention from containers:

```cpp
template <typename InputRange>
auto the_algorithm(InputRange&& input_range) {
	using uInputRange = std::remove_cvref_t<InputRange>;
	auto first = std::ranges::begin(input_range);
	auto last = std::ranges::end(input_range);
	// ... fancy algorithmic magic here
	return uInputRange(std::move(first), std::move(last));
}


int main () {
	std::u16string_view input = u"Hey there!";
	std::u16string_view result = the_algorithm(input);
	// shhh compiler, I do use the result...
	return std::ssize(result);
}
```

This did not compile at all.

First among the problems was that `std::basic_string_view<...>` did not have a constructor taking 2 iterators. And it wasn't alone: `std::span` and a whole lot of the `std::ranges` and [range-v3](https://github.com/ericniebler/range-v3) ranges did not have such constructors. On top of that, I had much more complicated algorithms where I wanted to take an `input_range` and an `output_range`, and return "how much did I advance in the range" back to the user. In the cases where users wanted to have a "sized output", the user would pass in a `std::span` or similar. If they wanted to basically have an "infinite output", they would use something like [range-v3's `ranges::unbounded_view`](https://ericniebler.github.io/range-v3/structranges_1_1unbounded__view.html). Of course, `std::span`, `ranges::unbounded_view` and other types in range-v3 didn't have the `iterator` + `sentinel` constructors either.

This posed significant usability problems for the APIs I was writing. I had to always return a `std::ranges::subrange<Iterator, Sentinel, std::ranges::subrange_kind::sized>` (or similar) to wrap the iterators I pulled out of the range. While generic, this type removes member functions and other nice things that the original types had and that someone on the non-generic end of my code would probably like to keep using!

When I initially made a pull request to range-v3 asking for one of the views (`unbounded_view`) to be changed so that it could work better when constructed from its `iterator`/`sentinel` pair, I was told that one-off fixes to singular types for the purposes of generic code were not sufficient. I needed to make sure that there was design integrity for adding the constructor to the type, or any type, in range-v3; that is, it should be based on a Concept with clear goals and benefits.

`reconstructible_range` is that concept:

```cpp
template <typename R>
concept pair-reconstructible-range =
	std::range<R> &&
	forwarding-range<std::remove_reference_t<R>> &&
	std::constructible<R, std::iterator_t<R>, std::sentinel_t<R>>;

template <typename R>
concept reconstructible-range =
	std::range<R> &&
	forwarding-range<std::remove_reference_t<R>> &&
	std::constructible<R, std::ranges::subrange<
		std::iterator_t<R>, std::sentinel_t<R>>
	>;
```



# Okay... But Why Does This Matter?

As I stated before, the generic code way to solve the problem above is to return a `std::ranges::subrange<Iterator, Sentinel, Kind>` from `the_algorithm`:

```cpp
int main () {
	std::u16string_view input = u"Hey there!";
	// std::ranges::subrange<
	// 	std::u16string_view::iterator, 
	// 	std::u16string_view::iterator, 
	// ...>
	auto result = the_algorithm(input);
	return std::ssize(result);
}
```

This allows a librarian to compose any 2 iterators into a generic range-like thingy. My problem with this is several things:

- it does not preserve any storage or layout preferences of the original range;
- it does not allow for a symmetric input range -> input range, output range -> output range API;
- it removes any familiarity that comes from the input or output types, including any convenience member functions or similar;
- and, it encourages stacking iterators on iterators, sentinels on sentinels, etc. to create far more type bloat.

By allowing a range to be reconstructible, we take the input type and return the input type, or the output range and return the output range: this is an inherently lossless operation that preserves any storage preferences the range itself has over the `iterator` + `sentinel` approach. It also allows me to write the above algorithm without resorting to `std::ranges::subrange`: it's no coincidence that I picked `std::u16string_view`. [P1629 - Standard Text Encoding](/_vendor/future_cxx/papers/d1629.html) is going to spend a lot of time writing algorithms that work over most input and output ranges: if people customizing the encoding objects need to return a `std::ranges::subrange<typename std::u16string_view::iterator, typename std::u16string_view::iterator>` for every single return where a `std::u16string_view` would be perfectly sufficient, I am not sure the people working with the lower level bits of my library will be very amenable to making their own encoding objects that do perform similar tasks.

Initially, I wanted `reconstructible_range` to be an exposition-only concept proposed for C++23, since it had no true effect on the current ranges interface as far as I could tell. Of course, I was to be given no such break: I got hit with the Corentin Effect™. The Corentin Effect is just the fact that when you want to propose a few minor things to fix up something in flight for the Standard -- e.g., Readable/Writable and Move-Only Iterators -- it turns out to have very big consequences that must be done now or risk losing them forever.

Reconstructible Ranges turned out to have somewhat similar implications.

While not as foundational as Move-only Iterators / Ranges, Reconstructible Ranges have type and operation optimization potential far beyond the API niceties I had first imagined. A paper I did not know about -- [Hannes Hauswedell's p1739 - Type erasure for forwarding ranges in combination with “subrange-y” view adaptors](https://stackedit.io/viewer#!url=https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2019/p1739r0.md) -- basically takes the idea behind Reconstructible Ranges and applies it to a handful of view adaptors where possible. This is highly beneficial in that it essentially solidifies the use case for a reconstructible range. We can generalize Hauswedell's work here by realizing that what he is trying to achieve -- for all of the changes _except_ the one to `counted_iterator` -- is essentially reconstructing the input range if it is at all possible.


# So What Now?

Seeing as this stuff has consequences for the standard, I wrote [D1664 - Reconstructible Ranges](https://thephd.dev/_vendor/future_cxx/papers/d1664.html) (to be published post-Köln). It creates a (exposition only) concept similar to the concepts written above that capture the intent and makes it applicable to far more range types and adaptors, and gives us a solid and concrete design foundation with which to move forward.

Speaking with Hannes and others about this, the design intent for ranges was that expressions would optimize inputs if at all possible. This is why `std::ranges::views::reverse(v);` will simply "undo" the `reverse_view` type wrapper if it detects that `v` is a view wrapped in a `reverse_view` type already. Hauswedell's paper codifies that into more views and ranges, and my paper provides the concept that his paper and several other papers -- including Corentin Jabot's and Casey Carter's [P1391](https://wg21.link/p1391) and [P1394](https://wg21.link/p1394) -- fix but do not provide any long-term, underlying guidance for.

To make it so that we need not have to individually justify each `iterator`/`sentinel`-pair or `subrange`-based constructor when we add it to the standard, we create this exposition only concept. This keeps us thinking about this for all the new ranges and range adaptors we add we add. It also ensures that adaptors in the future will do basic folding of the types to prevent template spew and type bloat that really grinds on our compilers today.


### Why Automatic Optimizations, though?

Others have expressed a concern that automatically optimizing such expressions would get in the way of use cases wherein users would like to generate a long Abstract Syntax Tree of range actions / adaptations. Primarily, this would allow them to look at the flow of operations, rather than have it all folded down into the final type of e.g. a `std::string_view`, `std::span` or `std::empty_view`. They would be more comfortable with a `std::ranges::views::simplify`, which would be applied in an expression and would attempt to simply all the operations after-the-fact:

```cpp
using namespace std::ranges;

std::vector v{1, 2, 3, 4, 5, 6, 7};
std::span v_ref(v);
// ranges::take_view<
//	ranges::drop_view<
//		ranges::drop_while_view<
//	std::span<int, std::dynamic_extent>
// >>>
auto unsimplified = v_ref | views::take(5) 
	| views::drop(2) 
	| views::drop_while(is_even_fn);
// explicitly simplify
std::span<int> simplified = unsimplified 
	| views::simplify;
```

This is an interesting idea. Doing so means the majority of the onus of simplification is on the "simplify" operation. I am less okay with this approach, however: it requires us to explicitly opt-in to something that _most_ users would want by default. People building abstract syntax trees could create a `std::ranges::subrange`-like type or adaptor that is not a _`reconstructible-range`_ and thusly prevent any kind of optimization: I would rather we tag the Domain Specific Language and cool-AST-case with extra markup, rather than the default usage requiring `| views::simplify` everywhere.

So, here we are. Trying to fix C++20 ranges before they head out the door!


### But the C++20 train already left?

I was told that this might be important enough for a National Body to make a comment. Seeing as it breaks source compatibility for ranges stuff, it very well might be. I am not a National Body, however, and I cannot make such comments! If someone feels like this is important enough and can file one on my behalf, I would be more than happy to fill out all the required paperwork, since I think this is important.



# And that's it!

Reconstructible ranges will become increasingly useful when writing range algorithms. Avoiding the return of iterator pairs and allowing an API to fluidly return someone the type that they put in is a huge API design win.

Hopefully it'll make it in, if not for C++20, for C++23! Toodle-oo, and thanks for reading. 💚
