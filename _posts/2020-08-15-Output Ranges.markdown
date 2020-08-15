---
layout: post
title: Throwing Out the Kitchen Sink - Output Ranges
permalink: /output-ranges
feature-img: "assets/img/2020-08-15/pexels-tiff-ng.jpg"
thumbnail: "assets/img/2020-08-15/pexels-tiff-ng.jpg"
tags: [C++, Ranges, ⌨️]
excerpt_separator: <!--more-->
---

Alternative title: "Stepanov is Correct, Part 54014673".<!--more-->

This post comes from the observation that despite having the power and flexibility of a range abstraction many of the `std::ranges` functions did not take output ranges, but output iterators. This was... disappointing, on _so_ many levels. I got into this when asking why we have this unsafe stuff with no alternatives in the standard, and I heard some... surprising answers. Some said that function-based "sinks" are better than output ranges, on top of issues handling non-view container types.

Let me not mince words, dear reader: functions ("sinks") are not only a pessimization of Stepanov's work, but do not belong as the first class citizen for output (or input) of a range-based API and everyone who said otherwise does not understand Iterators or Ranges.




# "Throwing More Gasoline on More Fires, Are We?"

Absolutely, dear reader! A small while back (an eternity in Pandemic Time™) I got into a [miniature twitter debate](https://twitter.com/thephantomderp/status/1253867585151606789) about how the `std::ranges` API commits the cardinal sin of being just as unsafe as its non-ranges counterpart despite having the technology to do better. This arose -- as you can see at the top of that twitter thread -- because the Standards Committee [demanded that we have an unsafe output iterator operation](http://eel.is/c++draft/format#functions-8) for `format_to_n`, and then the Standards Committee subsequently had a nice, long discussion about said operation being unsafe and dangerous. (Which, as a Committee Member, I got to 🍿 and watch as the e-mails rolled by.)

I'm surprised Victor Zverovich's forehead does not have a permanent palm imprint on it having to deal with that noise.

Given the title of this article it should surprise nobody that I think output ranges are the bees knees. Unfortunately, "we" -- as a collective library design consciousness -- apparently haven't all come to that conclusion? Alternatives to a typical `output_range` chiefly include a "different take" on this, by taking "sink" functions instead. I laughed to myself, because at the time I just sort of figured it was a foregone conclusion that it was a horrible idea.



# Narrator: Everyone Did Not Think Sinks Were a Horrible Idea

People are Very Seriously Considering this. Because of that Very Serious Consideration plus a question of "what happens with containers like `std::vector`", we did not use `output_range`s in the standard library despite [having the concept ready to go](https://eel.is/c++draft/range.req#range.refinements-1). `std::copy`, and all those functions that older versions of Visual Studio 2017 and below used to S C R E E C H about loudly when you didn't define `_SCL_SECURE_NO_WARNINGS`, take an input range and an output **iterator**. We, once again, decided not to deliver on the promise of a safer, better API because we had some hand wringing to do.

While there is a sense of familiarity and comfort that buffer overruns will continue to survive into the "modern" and "ambitious" language by API choice (and not just because The Language Hates You A Lot), let's get to the nitty-gritty: iterators and ranges -- particularly `output_range` -- are superior for three distinct reasons:

- Implicitly Memory Safe, with Explicit Opt-in
- Implicitly Allocation Safe, with Explicit Opt-in
- Optimize Better than Sinks Ever Can




# Implicitly Memory Safe with Explicit Opt-in

The problem with functions like `std::copy` are many, but primary among their security sins is the fact that they are "three legged" algorithms. Three Legged™ algorithms take 2 iterators for the input, and a _single_ (just one!) iterator for the output. It is _assumed_ that there is enough space in the output given the size of the input. As always, when you assume you make an `ass` out of `u` and `GOD WHAT IS THIS RANDOM FAILURE IN PRODUCTION--`

Ahem.

`std::ranges::copy` and all their friends don't improve here; the output target is still an output iterator. Did you pass in the iterator to a too-small buffer? Well, too bad: heck you, AND your heap/stack!

`[[segue]]` This past week, I got a question from someone learning C. They were gleefully calling `scanf` to read data when they had only allocated 4 `char`s of space with `malloc`. This was not enough space, and Visual C in return hit them with the "Heap Corruption" debug popup for their crimes. That we have APIs written for anno Domini 2020 that let you blow your leg off in the same way `scanf` does because there is no size information is an unsurprising and consistent tragedy, similar to Americans handling the number of gun-related mass death in their country:

thoughts & prayers for that lost leg, fellow programmer.

In lieu of not keeping the American politic in our standard APIs, let's [take this](https://eel.is/c++draft/alg.copy):

```cpp
template<input_­range R, weakly_­incrementable O>
	requires indirectly_­copyable<iterator_t<R>, O>
constexpr ranges::copy_result<borrowed_iterator_t<R>, O>
ranges::copy(R&& r, O result);
```

and make it safe with an output range:

```cpp
template<input_­range R, output_range O>
	requires indirectly_­copyable<iterator_t<R>, iterator_t<O>>
constexpr ranges::copy_result<borrowed_iterator_t<R>, borrowed_iterator_t<O>>
ranges::copy(R&& in_range, O&& out_range);
```


### And that's it.

Yes. By using an output range, we get safety from our API. To illustrate, we have some data:

```cpp
int destination_data[450];
int very_important_goddamn_integer = important_access();
int source_data[500]{};

/* ... */

std::span<int> source(source_data);
std::span<int> destination(destination_data);
```

We want to copy it. Here are the various ways we can copy it in C++20, and with the changes talked about above:

```cpp
// Old stuff, C++20:
// (0)
std::copy(source.begin(), source.end(), destination.begin());
// (1)
std::ranges::copy(source.begin(), source.end(), destination.begin());
// (2)
std::ranges::copy(source, destination.begin());
// New stuff, C++??:
// (3)
std::ranges::copy(source, destination);
// (4)
std::ranges::copy(source, std::ranges::unbounded_view(destination.begin()));
```

Function calls (0) through (2) will ride over your data and probably smash a few things on its way out. Maybe it will mess with `very_important_goddamn_integer`, too, that later gets used to loop over some data. Oops, there's a cute little vulnerability waiting to happen! If the data had been on the heap, it could have run over other heap structures or invade other data pools. Spurious failures occur, Heisenbugs, and more!

Now, contrast this to calls (3) and (4). (3) stops the copy operation early: it takes `destination.begin()` and `destination.end()`, and stops when the output is full or the input is exhausted. Did you intend to only copy 450 items? Probably not, but now we have not destroyed your heap, smashed your stack, or done other things that provoke Undefined Behavior. Wonderful!

(4) is a mouthful. "Why would anyone write that, it's so verbose and silly looking", some people might quip. But that's the beauty of this, dear reader: because I am explicitly deploying an `unbounded_view`, I can now explicitly mark every place in my code where I'm telling the API "blow my foot off". Now, instead of checking _every_ single call to `copy` in my codebase and frantically looking for where I've committed my cardinal sin (or waiting for the 14 hour Valgrind session to finish on my application that I have to write AutoHotkey scripts to click through because it's so damn slow), I can now just `grep` for all the places where I thought I was too cool for safety and check my assumptions. If it turns out to be one of those places: awesome! The code is self-documenting, too, which is equally attractive when I get hit by a giant yellow bus and you take over for me! You would know exactly WTF I was doing, whether or not I was too lazy to comment my code (or update my old, now-lying comments).

And as you fix my mistakes, I can smile up at you from Hell, my eternally burning form provided a brief respite as you are spared a massive debugging session. You did it, dear reader! ⚰️👍




# Implicitly Allocation Safe, with Explicit Opt-in

The other benefit of `output_range` (and, perhaps in the future, `output_view`) is their resistance to memory bloat. Remember that one of the salient objections to `std::ranges::copy(input, output)` is if `output` happened to be, say, a `std::vector<int>`. What is supposed to happen? There's 2 apparent choices here:

1. detect that we are working with a container rather than a plain "view" type: automatically wrap it in a `insert_at_end_view(output)`; or,
2. not give a damn, do no wrapping, pass it through as-is and just use `.begin()` and `.end()` and fill whatever is there, even if it's empty.

The answer to that question changes the meaning of the following code:

```cpp
int source_data[500]{};
std::span<int> source(source_data);
std::vector<int> destination{};
// insert into the vector, or do 
// `.begin()` and `.end()` and get an empty range?
std::ranges::copy(source, destination);
```

The answer I chose is neither (1) or (2): instead, I define a concept called `output_view`. We piggyback off the `view` concept, adding to it that it should be an `output_range` as well:

```cpp
template<class T>
concept output_view = std::ranges::output_range<T>
	&& std::ranges::view<T>;
```

This means that a ranged version of `std::ranges::copy` could accept only `output_view`-style types: in other words, things that are not containers. This lets us provide a clear error for the `std::vector` case, meaning we don't have to choose between either (1) or (2). By not having an "implicit push back", we fortify the algorithm against being implicitly allocation unsafe! Remember, in C++ all allocations may throw through the container and can leave us in unstable, or unanticipated, states. Forcing the use of specific kinds of ranges allows us to be far more clear, and thus communicate our intent clearly:

```cpp
int source_data[500]{};
std::span<int> source(source_data);
std::vector<int> destination_data{};
// very clear what I want out of my code here
try {
	std::ranges::unbounded_view destination(
		std::back_inserter(destination)
	);
	std::ranges::copy(source, destination);
}
catch(...) {
	// ...
}
```

Other types that could be made here are `std::front_inserter`, `std::push_at_middle_inserter`, `my::sorted_inserter`, or whatever else you wanted to use. This means, dear reader, that at no point can we "accidentally" pass in a `std::vector` or a `std::list` and suddenly explode our application with numerous allocations: we always get to make the explicit choice.




# Optimize Better than Sinks Ever Can

Yes. Sink-based APIs are underpowered. This is not the fault of trying to use sinks: on the contrary, it's because functions are too general-case for what `std::ranges` is trying to do. Let's take a copy example, but turn it into a source/sink style of API:

```cpp
template<class Source, class Sink>
Sink source_sink(Source source, Sink sink) {
	while (auto opt = source()) {
		sink(*opt);
	}
	return sink;
}

#include <optional>

int main () {
	int x[10000] = {0};
	int y[10000] = {0};
	
	auto source = [s=x, e=&x[10000]] () mutable {
		return s < e ? std::optional<int>(*s++) : std::nullopt;
	};
	auto sink = [d=y] (int v) mutable {
		*d++ = v;
	};

	// Use our source and sink
	source_sink(source, sink);

	return 0;
}
```

The assembly for the above code is identical to a while loop written with iterators (it [turns into a memcpy/memset](https://godbolt.org/z/qGryBP), and all is fine with the world). But,

what happens if we take that to its logical conclusion?


### Testing Sinks/Sources against Iterators


Functions can be both transparent, definition-visible objects like lambda expressions, or completely opaque DLL function calls the compiler cannot optimize. How does the compiler handle the simple case of "write the data" when put behind a function call's worth of indirection? I attempt to answer by crafting benchmarks of 4 categories:

- `direct` (e.g. `iterator_iterator_copy_direct`); represents when a literal `*a = *b;` is used to do the work.
- `inline` (e.g. `iterator_sink_copy_inline`); represents a lambda function written right next to the `copy` call to do the work.
- `transparent` (e.g., `source_sink_copy_transparent`); represents a struct with a visible definition in a header doing the work.
- `barrier` (e.g., `iterator_iterator_copy_barrier`); represents a compile-time barrier such as a DLL call doing the work.

I then apply this to 3 separate styles of writing the `copy_` code:

- `iterator_iterator` (Stepanov's concepts, iterators and iterators)
- `iterator_sink` (iterator sources, but sink function calls)
- `source_sink` (using 2 function calls)

The operation timed is a copy operation from one `std::vector<std::size_t>` to another `std::vector<std::size_t`. The source vector is generated from a random distribution, contains 10000 elements, and is the same between all benchmarks. Now, let's give it a try:

![Microsoft Visual C++ Benchmarks about Output Iterators and Ranges versus Sources and Sinks](assets/img/2020-08-15/vc++%20-%20Release%20-%20output%20range%20benchmarks%20data.png)

Okay, so the Stepanov iterator + iterator, direct write with `*destination_iterator = *source_iterator;`, remains the absolute best. Unfortunately, this benchmark is cheating because it's done on Visual Studio 2019 Release, x86_64, Release on an AMD Ryzen 3600. The cheating is because Visual Studio 2019's optimizer is, for some reason, complete garbage at optimizing these function calls, even when specifically passed special flags to ensure maximum performance and inlining (`/Ob3` and friends).

Let's look at GCC instead, given the same benchmarks, with `-O3`:

![GNU Compiler Collection C++ Benchmarks about Output Iterators and Ranges versus Sources and Sinks](assets/img/2020-08-15/g++%20-%20Release%20-%20output%20range%20benchmarks%20data.png)

"Aha!", they may exclaim. "Things are much more even, sources and sinks are just as good as iterators here! They're practically equivalent for most cases!" And indeed, they are equivalent! ... Except,

when they are not:

![GNU Compiler Collection C++ Benchmarks about Output Iterators and Ranges versus Sources and Sinks](assets/img/2020-08-15/g++%20-%20Minimum%20Size%20Release%20-%20output%20range%20benchmarks%20data.png)

Surprise! All we did was compile with `-Os` to save on size rather than `-O3`, and suddenly everything degraded and are not so equal anymore. `direct` remains the undisputed, untouched Queen of the pack while every other solution takes a penalty. Which, makes a lot of sense! Sources and sinks rely on function calls to communicate their values: this means that they are affected by what the compiler can do, or what the compiler can see. You know what's not hard to see through, dear reader?




# `T*`, `T*`.

That's right: pointers! Or, more specifically: `contiguous_iterator`s and `contiguous_range`s. Stepanov laid out groundwork that allowed us to achieve performance through direct writes by having direct access to the underlying data. Sources and sinks are fundamentally worse because they take what is a statically-deducible `memcpy` and turn it into a guessing game the compiler has to play with every object file. The GCC Release benchmarks above may be compelling, but remember that this is code written in the middle of a benchmarking API call with no other logic at all in the executable or surrounding code. Going from toy code to real business production logic is not the time to be taking gambles on how well your code plays Optimizer Bingo. But you don't have to take it from me, dear reader, take it from the author of the greatest C++ formatting library on earth:

> No, even if you don't (e.g. grow). I know it because we have a highly optimized sink-based format and a it's nowhere near {fmt} in perf. You really need direct writes. – [Victor Zverovich, `fmt` Author, April 25, 2020](https://twitter.com/vzverovich/status/1254030259109781504)

MSVC running in a circle, pissing itself, and then passing out in a drunken stupor for many of the benchmarks (on its best settings!!) should also be indicative here of "optimizer divergence". The only benchmark case that consistently delivered high-quality performance is the `iterator_iterator_copy_direct`, e.g. the direct writes Victor speaks of. I am not here to gamble with the compiler, forever writing [variations of this "peak", cursed memcpy](https://twitter.com/jfbastien/status/1288232681432440834) for the sake of performance. I am going to call `std::copy` or `std::ranges::copy`, and I am going to get my `memcpy` for free because Stephan T. Lavavej can write `if constexpr (_Is_contiguous_meow<_It, _DestIt>) { /* meowmcopy :3 */ }` and not depend on how the stochastic stream processor in my CPU or the optimizer feels today. My performance will not plummet because I turned on the [Jessica Paquette's amazing outliner to save code space](https://www.youtube.com/watch?v=yorld-WSOeU) for my Apple builds. I can have speed. I can save space. And, you know what else?

I can also have Sources and Sinks, without needing to butcher Stepanov's foundational work.


# What?

Yes. Sinks and Sources are strictly underpowered compared to Iterators/Ranges, because they are a strict superset of the functionality iterators and ranges bring to the table. Take a look at [the code of the benchmarks](https://github.com/ThePhD/phd/tree/main/benchmarks/output_range/source), in particular both the `iterator_iterator.cpp`'s `inline` benchmark, and the `source_sink.cpp`'s benchmark:

```cpp
// iterator_iterator_copy_inline benchmark, loop core
	auto sink_fn = [d = destination_data.begin()](std::size_t i) mutable {
		*d++ = i;
	};
	fn_output_iterator<decltype(std::ref(sink_fn))> sink(std::ref(sink_fn));
	std::copy(source_data.cbegin(), source_data.cend(), sink);

// source_sink_copy_inline benchmark, loop core
	auto source = [s=source_data.begin(), e=source_data.end()]() mutable {
		if (s == e)
			return tl::optional<std::size_t>();
		return tl::optional<std::size_t>(*s++);
	};
	auto sink = [d = destination_data.begin()](std::size_t i) mutable {
		*d++ = i;
	};
	source_sink_copy(source, sink);
```

They look almost entirely identical, because they are.

![Always has been meme of Ranges and Sinks.](/assets/img/2020-08-15/It's%20All%20Ranges.png)

Input ranges and output ranges __are the sources and sinks we've wasted so much time talking about__. With a simple "input function iterator" and "output function iterator" adaptation, you can write exactly the source/sink code in the iterator version. This means that Stepanov already had room for the function-based, source/sink world people have been asking about. Sources and sinks are just detrimental and regressive versions of `input/output_iterator` (and their range counterparts). That anyone claimed it as a "better" foundation for the Standard Library is 🍌🍌.


### Some Extra Tidbits

While we're here, check out these graphs of GCC in Release, and GCC in Release built with LTO (from a friend's beefy computer, not mine):

![GCC Release mode vector copy benchmarks.](/assets/img/2020-08-15/g++%20-%20Friend%20-%20Release.png) ![GCC Release mode vector copy benchmarks, with LTO on.](/assets/img/2020-08-15/g++%20-%20Friend%20-%20Release%20LTO.png)

Turns out, LTO makes the performance worse and adds a few microseconds to almost every functions runtime. (Sorry about the fact that the graphs look nearly identical here. I automatically size the graphs based on the data spread, but if you read the x-axis it turns out LTO gives nearly every function some sort of penalty, which is kind of awful!)

This all brings me to my final point. If you were just here to go on this journey with me and learned something (like to not trust MSVC's optimizer), then you can stop reading here, dear reader, because the rest of this is absolutely not required reading. But for damn sure, it is required writing on my part given what I have had to invest in this article and many other things related to C++ Standardization.




# You, Dear Reader, May Depart Now

Thanks so much for sticking with me until this point! I'm sorry that we essentially went in one giant circle just to arrive back at "Stepanov was right, yo", but sometimes it do be like that. I'll be working on more things over the coming months and hopefully they will be worth it to you, dear reader!

I'll catch you on the flip-side, ta-ta for now! 💚~

...

And now...

... Now, that they have left, that leaves us. Myself, and _You_. Not the "you" that is my dearest, most precious reader, whose libraries I want to see flourish and whose applications I want to see bloom in this world alongside their siblings, a meadow of creativity and realized potential. No... It leaves me with the Royal You,

the Committee.

First of all, how dare you?

Alexander Alexandrovich Stepanov is retired, not dead, and this is what you slide into the Committee with? In what universe did you think you could suggest a fundamentally new and wholly different foundation for the STL, do no research into the alternative that you presented, and then start planting the seeds of doubt in every person who wondered where `output_range`'s usages were in the Standard Library? How could you with such hubris and pomp declare that there exists a better basis than Stepanov's crowning C++ achievement with no research or development to back that up? There isn't even a grave for him to roll in: he can read these, too.

How dare you! You make these suggestions with a straight face and send me, a person gullible and naïve enough to believe you as an authority on the subject, running around and digging and research this "new" and "important" design? You present sinks and sources as an equal and deliberate reason when I ask as to why output ranges might not be credible or noteworthy or long-lasting in their abstractive power. And THIS is what I find, in a single sleepless night's worth of work?!

How much more could I fill an ENTIRE C++ paper with, over this shoddy construction of an idea? How much Committee member time do I need to waste making them read that paper -- to listen to it and to present it and to vote on it -- as if our time is not limited and it will not stop us from doing other things?

How many times am I going to go in bright eyed and bushy tailed, believing the words that come out of your mouth? How many times will it [turn out to be a lie wherein none of you sat down and researched the consequences](/to-bind-and-loose-a-reference-optional) of your suggestions and actions?

You are the Committee. Your words have weight, people will remodel codebases in pursuit of the ideals you set forth. When you say "pause", [engineers take heed and caution](http://lists.llvm.org/pipermail/cfe-dev/2018-July/058448.html) because they trust you.

I trusted you.

Every time I dig into a paper, or take a suggestion from the Committee's meetings out into the world, I expect that the Apex Library Designers Of Our Time Who Speak So Confidently Of This Idea are onto something. I am routinely disgusted to find that even the most basic benchmarks and litmus tests -- like the ones presented in this article consisting of even the simplest case ("copy a `vector` to another `vector`") -- fail. The most basic design philosophies do not hold, the code does not survive under time, space, or optimizer constraints, and the design is measurably and quantifiably worse than something someone else did X decades ago.

You have the distinct privilege of being a Committee Member. Many on this Good Green Earth are not lucky to make it or go. Some have to be [pulled from wherever they are in the world through the help of a literal non-profit organization](https://isocpp.org/about/financial-assistance-policy) just to get to Committee Meetings. Now, they are virtual, but instead the "plebeians" need to be "cleared" by an existing ~~proletariat~~ Committee member to join. You have no such hoops to jump through, you have access to all the history, you have access to all the e-mails. The history -- no matter how spotty and old -- is at your fingertips.

Yet, these are the things you hold up progress for? These are the lofty ideas that stop `output_range`s in their tracks, that force Victor to add an unsafe function, and then have to cycle back through a few days of WG21 internal derping about safety?

How DARE you waste THE most precious human resource -- our **time** -- during this critical pursuit of a better C++ and try to roll over Stepanov's work?

To [quote Victor](https://twitter.com/vzverovich/status/1254046921309614080),

> I'm kind of impressed that some WG21 people who presumably work on perf don't understand this.

Unfortunately, unlike Victor, I am no longer impressed. Please [start doing your homework](http://elementsofprogramming.com/eop_coloredlinks.pdf). Please run a benchmark. Please write a test. Please check your assumptions. Please do it before I have to check your work, and find out it doesn't hold. Please don't make me write another paper to counteract your bullshit and stop it from spreading in the C++ community. Please do the bare minimum.

The bar could not have been closer to the floor.

Don't make me pick it up again. 💚
