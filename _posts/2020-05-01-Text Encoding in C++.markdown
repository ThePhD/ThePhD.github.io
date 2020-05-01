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


# But First

The talk is already online (holy cow, Sy Brand and the Visual C++ team as FAST): 

<div style="text-align:center"><iframe width="640" height="480" src="https://www.youtube-nocookie.com/embed/w4qYf2pvPg4" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe></div>

In it, I describe what is essentially the fundamental basis operations for text encoding. Particularly, why a paper I wrote for Standardization -- P1629 -- chooses the 7 base required elements -- the "Lucky 7" -- to unlock all of the encoding operations. You can catch up on previous talks, current collective patron and sponsorship progress, papers, and more at the [portfolio page](/portfolio/text).


# Enabler: Concept-Driven Development

The most important takeaway from the presentation is by using the Lucky 7 design, one can enable perfect interop with the rest of the ecosystem at no cost to either the Standard Library developers or the end user. This means that anyone -- literally, anyone -- can write their own encoding and handle as much or as little as they want to, and get support that works in any part of their codebase, full stop. It also means that abstractions built on top of these objects which have the 7 basis pieces will continue to hold from now until the end of time.


## Resiliency: not optional for C++ Proposals

Given all of the talk around ABI, things that do not have the potential to scale until the end of time cannot survive in C++ where the Standard is concerned. While some people will postulate that there's room for growth and appetite for fixing potentially breaking things in C++, I will note that we do not act that way as a Committee at all.

For example, we voted to fix ABI where possible and consider proposals for ABI when necessary without voting them down as one of the first orders of business in the C++ February Prague 2020 meeting. Then, quite literally almost immediately after that vote, we [rejected a `std::regex` proposal (P1844)](http://www.akenotsuki.com/misc/srell/en/proposal/) that [had significant implementation experience](http://www.akenotsuki.com/misc/srell/en/) strictly on the basis of a single vendor's implementation being unable to change due to implementation detail ABI problems. This was after the author went to great lengths to prove source compatibility and demonstrate that the Unicode regex engine could only be applied to `char8_t`, `char16_t`, and `char32_t` to not have to deal with backwards compatibility issues for `char` and `wchar_t`. Instead, SG16 members are writing a paper to deprecate `std::regex`, so people stop asking us to fix it since we will, summarily, throw their concerns out the window for said ABI.

If something does not have resiliency built into the API, then that API is destined not to last in the C++ world, especially when it comes to the hyper-conservative push that almost all standard libraries are undergoing. For example, C++ Committees of the past decided to store `std::memory_resource*` as a pointer with explicit "reified" semantics, exposing the polymorphism in the interface and to the end user (see: [N3525](https://wg21.link/n3525)). In less than 3 years after Polymorphic Resource and Polymorphic Allocator standardization in C++17, that decision promptly pimp slapped us upside the head [when Providing Size Feedback in the Allocator Interface (P0401)](https://wg21.link/p0401) showed up. For exactly the same reasons why the `<iostream>` interface cannot be fixed (virtual, overridable calls in the public-facing interface), we cannot fix polymorphic allocators to do much the same without risking a hard ABI break for downstream users and thusly will actually leave any allocator fixes off from Polymorphic Allocators. (Oops.)

If an API is not designed to last into the future on a solid basis, it will not stand even a year, which is why this Part 3 presentation is the most important presentation on Text for C++ I could possibly give. It proves that this static interface supplies everything needed and, as explained in [Part 2](https://www.youtube.com/watch?v=FQHofyOgQtM), shows that the concept can be extended Without Loss of Generality to runtime decisions as well.

This is an immense win for both stability and extensibility, giving us a solid foundation through which encoding support can be realized in C++ without the majority of the back-breaking pains that ABI and API stability cause us and have been causing us for the last 20 years.


## Returning the means to the People

This is by far the most important part of the Concept-Driven, compile-time interface. Lucky 7 enables everyone to provide exactly what they want for their application or library, _without_ having to come hat-in-hand to the C++ Standards Committee begging for support for X encoding or Y bit of legacy. The encodings targeted for the standard will fully support Unicode Code Points as the swivel by which they interop with everyone else, and as long as there is a way to get from your encoding to _some_ form of Unicode Code Points, it works. As a non-exhaustive list, these are just some of the encodings for which this is possible. With over 50+ globally recognized encodings supported fully by Unicode, it's hard to not choose it as the fundamental pivot point.

Still, there are some pariahs. Defunct data which used private use area code points but are now fully integrated into Unicode, custom company encodings for very strange reasons, and more: there's lots of reasons why people have not switched fully to Unicode yet, including "we did not hire the engineers to do that migration of our 40 year old government database". Which is alright: as shown in the presentations for Parts 1, 2 and 3, the `code_point` and `code_unit` types are entirely customizable on encoding objects, meaning someone can define the specific collection of bridge encodings necessary to get from where they are, to Unicode.

Or not.

Nothing stops someone from creating an entirely self-consistent universe outside of Unicode using encoding objects. And the end-game for all of this that is shown in the CppCon 2019 Part 1 Presentation is perfectly fine with this too: `std::basic_text<my_exotic_weird_encoding>` still works. (You pay the price of not being able to easily play with the _rest_ of the standard library, but what could anyone expect us to do with a special rot13-encoded version of EBCDIC-US??)


# Okay, so Lucky 7. Is it fast?

The base encoding object will never be fast. But, that is not the point of the base encoding object. Lucky 7 provides you with full interop and correct-by-construction plug-and-play ability. Because it works by encoding and decoding only one full unit of information at a time, most SIMD-happy folk won't be able to do all the fantastic tricks they do to encode, decode, and validate data.

_And that's fine._

Dear reader, the word "Basis" is not used lightly in the presentation material; it is intentional. It is the **bog standard basic minimum** that anyone has to write to achieve their goal. This is intentional, because making the minimum requirements easy means higher adoption rates when fresh graduates/self-taught people are capable of writing their own encodings. This does not mean performance is left high and dry: on the contrary! There's a huge wealth of extra functions and customizations someone can engage in to speed up the things they care about most...

But that comes later, in Talk Part 4.

For now, you should know that I and others can provide the internal beta release of this work through the consultancy and contracting work of Shepherd's Oasis!

![Shepherd's Oasis, LLC Consulting Presentation Slide](/assets/img/2020-05-01/soasis-consulting-presentation.png)

If you need something done or some annoying bugs related to Unicode or Text or Encoding squashed in your code base, or you want to employ us for jobs around anything related to C, C++, Scripting on Embedded Devices, or more, drop us a line at [shepherd@soasis.org](mailto:shepherd@soasis.org). And, as usual, you can also just be a regular sponsor or a patron or a donator by going [here](/support), with the added benefit that private musings, API beta tests, and more are available to higher-tier patrons. To those of you already contributing:

Thank you, for helping me make C++ a better place.

Toodle-oo. 💚
