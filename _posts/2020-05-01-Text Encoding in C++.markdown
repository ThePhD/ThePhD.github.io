---
layout: post
title: Starting a Basis - Shepherd's Oasis and Text
permalink: /basis-shepherds-oasis-text-encoding
feature-img: "assets/img/2020-05-01/soasis.png"
thumbnail: "assets/img/2020-05-01/soasis.png"
tags: [C++, C, Text, Unicode, ⌨️, 📜]
excerpt_separator: <!--more-->
---

Just yesterday, I gave a talk at the recent Microsoft Pure Virtual C++ 2020 Conference and wanted to expand on a few things I said, since there were a lot of questions!<!--more-->


# But First - The Talk Itself!

The talk is already online (holy cow, Sy Brand and the Visual C++ team as FAST):

<div style="text-align:center"><iframe width="640" height="480" src="https://www.youtube-nocookie.com/embed/w4qYf2pvPg4" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe></div>

[Slides are here](/_presentations/unicode/Pure%20Virtual%20C++/2020/Lucky%207%20–%20Designing%20Text%20Encodings%20for%20C++.html).

In it, I describe what is essentially the fundamental basis operations for text encoding. Particularly, why a paper I wrote for C++ Standardization -- [p1629](https://thephd.dev/_vendor/future_cxx/papers/d1629.html) -- chooses the 7 base required elements -- the "Lucky 7" -- to unlock all of the encoding operations required for an end-user. You can catch up on previous talks, current collective patron and sponsorship progress, papers, and more at the [portfolio page](/portfolio/text).




# Lucky 7 - Enabling the Universe

The most important takeaway from the presentation is by using the Lucky 7 design, one can enable perfect interop with the rest of the ecosystem at no cost to either the Standard Library developers or the end user. It also allows someone to write all the necessary encoding and decoding operations, without having to write some specific code for each one, including:

- Transcoding ("go from A to B en-masse")
- Encoding ("go from A's code points (unicode code points), to A's code units")
- Decoding ("go from A's code units, to A's code points (unicode code points)")
- Validation ("is this text validly-encoded text, are these code points valid to be transformed to these code units")
- Counting ("how many code units / code points will this result in")

This means that anyone -- literally, anyone -- can write their own encoding for the Lucky 7, and get support that works in any part of their codebase **full stop**! It also means that abstractions built on top of these objects which have the 7 basis pieces will continue to hold from now until the end of time.




## The End of Time? Really?

Yes, really!

Given all of the talk around ABI, things that do not have the potential to scale until The End Days™ cannot survive in C++ where the Standard is concerned. While some people will postulate that there's room for growth and appetite for fixing potentially breaking things in C++, I will note that we do not act that way as a Committee at all.


### For Example...

We voted to fix ABI where possible and consider proposals for ABI when necessary without voting them down as one of the first orders of business in the C++ February Prague 2020 meeting. Then, quite literally almost immediately after that vote, we [rejected a `std::regex` proposal (P1844)](http://www.akenotsuki.com/misc/srell/en/proposal/) that [had significant implementation experience](http://www.akenotsuki.com/misc/srell/en/) strictly on the basis of a single vendor's implementation being unable to change due to implementation detail ABI problems. This was after the author went to great lengths to prove source compatibility and demonstrate that the Unicode regex engine could only be applied to `char8_t`, `char16_t`, and `char32_t` to not have to deal with backwards compatibility issues for `char` and `wchar_t`. Instead, SG16 members are writing a paper to deprecate `std::regex`, so people stop asking us to fix it since we will, summarily, throw their concerns out the window for said ABI.

If something does not have resiliency built into the API, then that API is destined not to last in the C++ world, especially when it comes to the hyper-conservative push that almost all standard libraries are undergoing. For example, C++ Committees of the past decided to store `std::memory_resource*` as a pointer with explicit "reified" semantics, exposing the polymorphism in the interface and to the end user (see: [N3525](https://wg21.link/n3525)). In less than 3 years after Polymorphic Resource and Polymorphic Allocator standardization in C++17, that decision promptly pimp slapped us upside the head [when Providing Size Feedback in the Allocator Interface (P0401)](https://wg21.link/p0401) showed up. For exactly the same reasons why the `<iostream>` interface cannot be fixed (virtual, overridable calls in the public-facing interface), we cannot fix polymorphic allocators to do much the same without risking a hard ABI break for downstream users and thusly will actually leave any allocator fixes off from Polymorphic Allocators.


### Oops?

A big oopsie kapoopsie indeed, dear reader!

If an API is not designed to last into the future on a solid basis, it will not stand even a year (let alone the 3 between standards revisions, as shown above). Static, concept-based interfaces that do not involve specifically-sized structures where reasonable and specific connectivity points have proven invaluably useful. The reason Stepanov's iterator library and its improvement -- C++ Ranges -- hold to this day is because of the solid basis operations upon which they were built, informed by the algorithms they are used for.

This is why this the Pure Virtual C++ Part 3 presentation is the most important presentation on Text for C++ I could possibly give. It proves that this static interface supplies everything needed and, as explained in [Part 2](https://www.youtube.com/watch?v=FQHofyOgQtM), shows that the concept can be extended Without Loss of Generality to runtime decisions as well with some small and elegant tweaks on the library designer's (the standard library's) end of the bargain.

This is an immense win for both stability and extensibility, giving us a solid foundation through which encoding support can be realized in C++ for the next 40 years without the majority of the back-breaking pains that ABI and API stability have caused us for the last 20 years.




# Okay, but why all this flexibility?

Ultimately, it's for you and I, dear reader.

This is by far the most important part of the conceptual (as in Stepanov, not as in C++), compile-time interface. Lucky 7 enables everyone to provide exactly what they want for their application or library, _without_ having to come hat-in-hand to the C++ Standards Committee begging for support for X encoding or Y bit of legacy. The encodings targeted for the standard will fully support Unicode Code Points as the swivel by which they interop with everyone else, and as long as there is a way to get from your encoding to _some_ form of Unicode Code Points, it works. With over 50+ globally recognized encodings supported fully by Unicode, it's hard to not choose it as the fundamental pivot point.

Still, there are some pariahs. Defunct data which used private use area code points but are now fully integrated into Unicode, custom company encodings for very strange reasons, and more: there's lots of reasons why people have not switched fully to Plain Ol' Unicode yet, including "we did not hire the engineers to do that migration of our 20 year old government database". Which is alright: as shown in the presentations for Parts 1, 2 and 3, the `code_point` and `code_unit` types are entirely customizable on encoding objects, meaning someone can define the specific collection of bridge encodings necessary to get from where they are, to Unicode.

Or not.

Nothing stops someone from creating an entirely self-consistent universe outside of Unicode using encoding objects. And the end-game for all of this that is shown in the CppCon 2019 Part 1 Presentation is perfectly fine with this too. `std::basic_text<my_exotic_weird_encoding>` still works. You pay the price of not being able to easily play with the _rest_ of the standard library, but what could anyone expect us to do with a special rot13-encoded version of EBCDIC-US? Your code will work in general but any further additions of Unicode to the standard library -- Normalization, Segmentation (Grapheme Clustering), Collation and more -- won't work without a bridge to Unicode.

This means that rather than trying to bludgeon everyone and force them to use Unicode, we instead highly incentivize it so users can get the goodies they need by participating in a robust ecosystem. This is a win for the common user who:

- wants to live in a Unicode-enabled world;
- add bridges to get their outsider-encodings into the Unicode world in a way C++ did not previously support well;
- and, give people who have come up with weird but entirely valid data representations to keep doing what they are doing.

There are guard rails on this road and it is well-trodden, but we give you the power and the blessing to go veer off through your 40 day & 40 night desert journey.

In short, it is not this abstraction's place to judge you, but enable and empower you, dear reader!




# Well that's nice of Lucky 7. But is it fast?

The base encoding object will never be the fastest you can do. But, that is not the point of the base encoding object. Lucky 7 provides you with full interop and correct-by-construction plug-and-play ability. Because it works by encoding and decoding only one full unit of information at a time, most SIMD-happy folk won't be able to do all the fantastic tricks they do to encode, decode, and validate data.

_And that's fine._

Dear reader, the word "Basis" is not used lightly for Lucky 7; it is intentional. It is the **bog standard basic minimum** that anyone has to write to achieve their goal. Making the minimum requirements easy means higher adoption rates. It means fresh graduates/self-taught people are capable of writing their own encodings without having to read several presentations: they just need to see a few code samples. This does not mean performance is left high and dry: on the contrary! There's a huge wealth of extra functions and customizations someone can engage in to speed up the things they care about most...

But that comes later, in Talk Part 4 of 5.

For now, you should know that I and others can provide the internal beta release of this work, or a hand-crafted version for you, through the consultancy and contracting work of Shepherd's Oasis!

![Shepherd's Oasis, LLC Consulting Presentation Slide](/assets/img/2020-05-01/soasis-consulting-presentation.png)

If you need something done -- some annoying bugs related to Unicode or Text or Encoding squashed in your code base, employ us for jobs around anything related to C, C++, Scripting on Embedded Devices (Lua or otherwise), and more -- drop us a line at [inquiries@soasis.org](mailto:inquiries@soasis.org) or visit [our website](https://soasis.org).

If you are just an individual, you can also just be a regular sponsor or a patron or a donator by going [here and selecting an option that works for you](/support), with the added benefit that private musings, API beta tests, and more are available to higher-tier patrons. To those of you already contributing: thank you, for helping Shepherd's Oasis make C++ a better place. And for those of you yet to come,

we look forward to making your life easy and painless in the otherwise brutal C++ landscape.

Toodle-oo! 💚
