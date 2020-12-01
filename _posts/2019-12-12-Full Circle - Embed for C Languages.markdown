---
layout: post
title: Going Full Circle on Embed in C++
permalink: /full-circle-embed
feature-img: "assets/img/2019-12-12/assorted-color-stained-glass-spiral.jpg"
thumbnail: "assets/img/2019-12-12/assorted-color-stained-glass-spiral.jpg"
tags: [C++, C, embed, Circle, 📜]
excerpt_separator: <!--more-->
---

Is consumption of content high in irony dangerous? In this post, we find out!<!--more--> Particularly, for a long time, I have been promising to write about `std::embed`. Many people want to know where it is or what I am doing with the C++ paper, p1040, and how it fares in the C++ Committee. So...


# Let's Talk About It

`std::embed` is [a proposal for a magical library](/vendor/future_cxx/papers/d1040.html) function that will take a "resource name" and load it up for use at compile-time. It was conceived over a year ago with the help of Lilly (LilyPad), Arvid Gerstmann, Nicole Mazzuca, Agustín Bergé and a few others. It looked like fanciful -- but simple -- magic back then, but a year later I found out that `std::embed` is very real. I know this not as a matter of faith and the heart, but of carnal physicality. That is, because I

_[made it real](https://godbolt.org/z/HLTuci)_ with my own two goddamn hands.

Weeks of carving through the compiler against the advisement of several people ("Don't implement it in GCC"; "Why do you like things being hard"; "Stop it and just do Clang if you must"; "It's trivial to implement having an implementation is a waste of time") and [2 CppCast episodes later](https://cppcast.com/jeanheyd-meneide-unicode/), I had in my hands `std::embed`. And not just that: a small detour through the [WG14 Ithaca, NY October meeting](/follow-the-river-wg14-ithaca-2019) gave me a glimpse into C and hope that they could use a little extra help too. I finished the earliest implementation of `#embed` for the C language, so our sister language could benefit in the same ways C++ would from `std::embed`. Finishing it during the October 2019 C meeting meant that I was prepared for the upcoming C++ meeting.


# Just in Time

I finished it for the November 2019 Belfast C++ Standards Meeting. I was overjoyed: this timing meant that lulls in the NB Comment processing could allow for P1040 to be reviewed. At the request of Hana Dusíková and her eternal wisdom, I created a presentation for it overnight. The goal was to present to Evolution Working Group (EWG) - the main Language Design group of the C++ Committee - and Study Group 7 (SG7) - the Compile-Time Programming group - during the week and get their ideas.

[Equipped](/_presentations/standards/C++/2019%20November%20Belfast/p1040/P1040%20-%20std%20embed.html) as best I could with Wording, Implementation, and fixes for various complaints, I did my presentation first in EWG.

It went well.

By providing both `#embed` and `std::embed` in the same paper and proposing both as a way to work with both recursive resources (shaders with `#include`, for instance) and also help static dependency discovery (with `#embed`), I quelled much of the room's concerns save a few. The final votes turned out as follows:

> EWG is interested in hearing more of `#embed` and `#embed_str` in the general direction.
> 
> SF| F|N|A|SA
>  3|11|2|1| 2

> EWG is interested in hearing more of `std::embed` in the general direction presented today.
> 
> SF| F|N|A|SA
>  3|11|2|1| 2

That is strong direction to move forward for the C++23 timeframe. With this, I had grounds to come back to EWG after answering a few more questions and ironing things out (I already had the wording written, and so I was fairly happy since it would be at most minor adjustments and implementation tweaks).

It was time to go to the next room.


# SG7

It was SG7's turn to see the paper. While Study Groups are more or less hierarchically beneath the main groups (EWG, LEWG, Library Wording and Core Wording), their recommendations hold potent sway. Considering exactly who was in the room for SG7 -- several powerful EWG regulars who could not be there for the main discussion -- getting approval from SG7 was critical to surviving Round 2 for C++23 time in EWG.

It didn't work out like that:

> Do we want more generic API for such use-case?
> 
> SF|F|N|A|SA
>  4|4|3|1| 1


### Uh. "Generic API"?

![Confused man meme.](/assets/img/2019-12-12/Confusion.jpg)

You may be wondering what this question actually means, and what the hell a more "generic API" is.

Well, I was too.

The general idea is that `std::embed` (and `#embed`) were hyper-focused ideas and only solved a "narrow use case". A certain meta-language I had heard of but did not have the time to play stirred amongst meta-programming enthusiasts in the Committee, and that enthusiasm was spilling into wanting to change the current compile-time programming direction. The new language brewing up this change is [Circle](https://www.circle-lang.org/), by Sean Baxter.

Promising the use of all of C++ at compile-time -- and not as part of AST computation using `constexpr` -- Circle set itself up as the new, awesome way to think about compile-time programming. It was founded on the idea that just running regular, right-proper C++ and having explicit demarcation when going from compile-time to run-time was a thing. A handful of other proposals got bodied in SG7 from this new way of thinking. Folks emphasized we look into something Circle-like for our reflection needs, rather than go with the current way forward. `consteval` variables and the constant evaluation side effects model got stalled, a "new direction for Reflection" was mentioned in relation to developing this Circle-like meta-language, and of course

`std::embed` and `#embed` got hit by the bus.

After all, why have `std::embed` or `#embed` if you can just use `std::fstream` or `fprintf`/`FILE*` in an `@meta` context and then dump that data into C++? "Too specific", "too dangerous", "macros are crap", "why is `std::embed` not pulling from a static file pool with directives"; the list of reasons went on as I failed to defend P1040 properly against WG21's SG7.

And so now it lays quietly in critical condition, while we wait to see if this new Circle-like direction for compile-time programming comes to play. Unfortunately...


# Patience is not a Virtue

I battled the betentacled Necronomicon of GCC, with files that have Copyright notices dating back to the 70s as I shaped my own preprocessor directives and gnashed my teeth in pain with the C and C++ parsers. I cut myself on the Vorpal Blade of Clang's Constant Expression Evaluator as I struggled to understand the specific arcana to invoke the preprocessor and built-ins correctly. I fielded e-mails from National Labs employees, spoke shop with static analysis tool vendors, handled Twitter posts from Game Developers, and responded to DMs from embedded developers wondering why their requests were being ignored and their desires denied.

Patience is not a virtue for what should have been a 40 year old feature.

If SG7 was going to force us to consider Circle, then by all means it was time to consider Circle myself. I set up a few tests, and took the greatest litmus test among them - embedding 50 MB of data in an executable - to give Circle a spin.


### Speed and Space is Everything

One of the chief goals of `#embed` and `std::embed` was to solve 2 very important problems (asides from ease-of-use) for people working with large binary data in C++:

- do not take 20-40x additional memory to compile large brace-delimited sequences of numbers;
- and, do not take exponential time to parse, handle and work with the initializer lists generated by large brace-delimited sequences of numbers.

There were also other minor goals, such as allowing static analysis tools to retain file and location information without needing the build system to understand where data came from. This kind of information is helpful rather than just getting what _happens_ to be a particularly large array in C++ and having no idea about file identity.

To give you an idea of the disparity between the speed of the implementations, here are some timing numbers from a few methods of embedding a 50 MB blob I intend to do compile-time things with in GCC.

|         Methodology        |    Time     |
|----------------------------|-------------|
| `#embed`                   | 17 seconds  |
| `std::embed`               | 16 seconds  |
| `xxd`-generated `#include` | 621 seconds |

Yes, you can pay a 36x (or worse) time penalty for using `#include`-based binary data.

During this time, `#embed` and `std::embed` took almost no memory overhead, while the `xxd` method happily gargled **gigabytes** of my memory to get its job done and slowed things to a crawl on my less powerful machines. This was even with special code for `#embed` to survive tooling that sits between the Preprocessor and Real C++ Processing™ (those details will be a separate blog post). With these timings, I then went to speak with the Developer of Circle so I could give the meta-language its fair shake:

> @seanbax Hi!
> 
> I wanted to vomit out a large amount of static data that I read in from an fstream at compile-time using circle. I wanted to have that data available for some constexpr calculations. What is the most optimal way to do this in Circle for a 40-50 MB file?
> – Me

The twitter thread continues, with him discovering slowly what I was doing and why I was doing it. Lo and behold, the conclusion Sean Baxter comes to:

> I'll add a new @embed keyword that takes a type and a file path and loads the file and embeds it into an array prvalue of that type. This will cut out the interpreter and it'll run at max speed. Feed back like this is good. This is super low-hanging fruit.
>  – Sean Baxter

...


# Oh. My. God.

![Picard Facepalm](/assets/img/2019-12-12/Picard Facepalm.jpg)

This is not in criticism of Sean Baxter's response. If anything, I am overjoyed he is adding the keyword, because the early build timings were not doing Circle any favors (I offered to hold off on publishing timings and to wait for the `@embed` keyword, and he accepted).

This is a glaring WTF, aimed right at SG7.

Told I was being too specific, told I should not use macros, told I should make a more generic API and told that I was potentially trying to [cram things through the Committee for the sake of having my favorite feature](https://lists.isocpp.org/sg7/2019/11/0015.php), my irony intake for the rest of the year hit ABSOLUTE CAPACITY.

The language I was told to look for to guidance -- Circle -- was adding a feature **exactly like `std::embed`** into it to handle **exactly the use cases I outlined in my paper**...

...


AAAAAAAAAAAAAA-


# AAAA**AAAH**-- !!

Is this what hell is like? Am I forever condemned to roll this enormous boulder up the same hill side, over and over again, only to slip and have it roll down to the base of the hill so that I must force myself to start all over again?

What is even more hilarious is that Sean Baxter also has another form of this (undocumented) in Circle called `@array`, which takes a pointer + a size and turns that into an array for quick, spiffy use at runtime. Twice over, Circle has implemented exactly what I am trying to get into C++. Because C++ was so absolutely and utterly **bad** at handling large array literals and binary data, he worked something in to make simple things simple and not require a separate build step, like almost EVERY cross-platform solution -- Qt included -- does.

So, I froze P1040. Even if I walked down the Circle Language path, I would end up right back here eventually the same way Sean Baxter did, so why bother doing that? Ending up hitting this circular reasoning in WG21 C++, pun **absolutely** intended, bothers me perhaps a little more than it should. I will be taking just `#embed` to WG14 C. When I get there, I will have to deal with WG14's general aversion to new language features that are not EXTREMELY well-motivated. Which is still frustrating, but at least it makes sense and is understandable and I can adequately explicate a case in front of that Jury when the time comes.

Maybe SG7 might change their minds on this soon. I don't know. But until that happens, I won't be running in any circles over it.

Later-taters. 💚
