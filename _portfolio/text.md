---
layout: post
title: Text for C++
feature-img: "assets/img/pexels/adult-art-asia-engin-akyurt.jpg"
img: "assets/img/pexels/adult-art-asia-engin-akyurt.jpg"
date: May 21st, 2021
tags: [C++, Unicode, Text, 🚌, ⌨️]
---


There does not exist a good, flexible, backwards-compatible solution for Text in C++. Choosing a library either requires taking the entire thing (Qt), getting involved in a very complex interface (ICU), dealing with sometimes limiting API choices (CopperSpice), or having a well-done but very opinionated design set (the proposed-for-Boost Boost.Text library).

Where is the standard library-friendly, maximum performance solution for handling text encoding and decoding in C++ and C?

This project is the push to reach that goal.




# Publicly Available Implementation

The Publicly-Available Implementation is [here: https://ztdtext.rtfd.io](https://ztdtext.rtfd.io). You can track progress on this page, through the documentation's ["Progress & Future Work" section](https://ztdtext.readthedocs.io/en/latest/future.html), or at the [the GitHub Repository](https://github.com/soasis/text).

[![Liberate your text using the ztd.text library.](/assets/img/portfolio/ztd.text.documentation.png)](https://ztdtext.rtfd.io)

The C Library implementation — Cuneicode — will be made publicly available as funding, scholarship, and sponsorship goals are reached.




# Current Funding

Funding goes toward:

- Funding development;
- Targeting specific features;
- Covering general library support;
- Covering specific company or vendor support;
- and, Attending WG14 (C Committee) and WG21 (C++ Committee) meetings.


Specialized solutions for C++11 (or C++03) can be made. If you, your company or organization is interested in helping or need special features/early access to features listed below, please [get in touch with these folk through their website](https://soasis.org/contact/opensource/) or [by e-mail](mailto:inquiries@soasis.org).



## Funding Goals and Progress

Below are the published funding goals. Sponsors may pay into specific goals or, if given a large enough donation, create a new goal entirely; otherwise, funding falls into the categories in a top-to-bottom, linear fashion. Goals marked (_Stretch_) are not quite bare-minimum necessary, but would be absolutely wonderful to accomplish!

- [🎊 Accomplished!] Bootstrap Initial Development, to get library tested and released;
- Normalization Forms and C-based Span Implementation (Cuneicode C Library)
- WHATWG and CJK Encoding Tests
- Cover C Standard Library development to reach maximum amount of users with basic functionality;
- Reach Full-Time Text Development to reach 2022 Goal;

_Current Goal: Normalization Forms and C-based Span Implementation_

Current Goal Total: $1,275.55 USD / $20,000.00 USD

[ ⣿⣿⣤⣀⣀⣀⣀⣀⣀⣀⣀⣀⣀⣀⣀⣀⣀⣀⣀⣀⣀⣀⣀⣀⣀⣀⣀⣀⣀⣀⣀⣀⣀⣀⣀⣀⣀⣀⣀⣀⣀⣀⣀⣀⣀⣀⣀⣀⣀⣀ ]




# Technical Details

The work is ongoing. The latest public documentation for the released library can be found [here: https://ztdtext.rtfd.io](https://ztdtext.rtfd.io).

The C++ library submodules and builds on top of the C one for fast-path functions. Internally, the C library is implemented with C++ and -- hopefully soon in the future -- vectorized by hand or with [SIMD/`std::experimental::simd`](https://en.cppreference.com/w/cpp/experimental/simd/simd). Document trails:

- [Latest Working Draft, C++](/_vendor/future_cxx/papers/d1629)
- [Latest Working Draft, C](/_vendor/future_cxx/papers/C%20-%20Efficient%20Character%20Conversions)


The principles and inner workings of the implementation are detailed in a series of talks, slides and posts:

0. !!Con 2021
  _😱 Oh, No! 😱 The Lowest-level‡ Programming Language is Unicode-aware and I have no excuses?!_  
  May 20th, 2021  
  Virtual Conference  
  - [Video](https://www.youtube.com/watch?v=eqtZuHveQMU&t=508s)
  - [Slides](/_presentations/unicode/!!Con/2021/Oh%20No%20Unicode.html)

0. C++ on Sea 2020
  _🤿 Deep C Diving - Fast and Scalable Text Interfaces at the Bottom 🤿_  
  July 16th, 2019  
  Virtual Conference  
  - [Video](https://www.youtube.com/watch?v=X-FLGsa8LVc)
  - [Slides](/_presentations/unicode/C++%20On%20Sea/2020/Deep%20C%20Diving.html)

1. C++ Russia Moscow 2020  
  _🏎 Burning Silicon - Speed for Transcoding in C++23_  
  June 30th, 2020  
  Virtual Conference  
  - [Video](https://www.youtube.com/watch?v=RnVWON7JmQ0)
  - [Slides](/_presentations/unicode/C++%20Russia%20Moscow%20Online/2020/Burning%20Silicon%20-%20Speed%20for%20Transcoding.html)

2. Pure Virtual C++ 2020  
  _Lucky 7 - Designing Text Encodings for C++_  
  April 30th, 2020  
  Virtual Conference  
  - Abstract: Text handling in the C and C++ Standards is a tale of legacy encodings and a demonstration of decisions made that work at the moment don’t scale up to the needs of tomorrow. With Unicode on the horizon, C++20 prepared fundamental changes such as char8_t and polishing a things to make it easier to catch bad conversions and logical program errors when working with encoded text. Still, the landscape has poor support for transcoding from one encoding to the other, let alone talking about higher level algorithms such as how to compare two text forms which render identical to the user but have different bit patterns. This talk explores the fundamental design space behind Encoding, Decoding and Transcoding text. It describes the benefits of the API under active consideration of text, potential speed gains from such an API, and how it enables better handling of complex tasks such as normalization.
  - [Video](https://www.youtube.com/watch?v=w4qYf2pvPg4)
  - [Slides](/_presentations/unicode/Pure%20Virtual%20C++/2020/Lucky%207%20–%20Designing%20Text%20Encodings%20for%20C++.html)

3. Meeting C++ 2019  
  _Catching ⬆️: Unicode for C++ in Greater Detail - 2 of 5_  
  Saturday, November 16th, 2019  
  Berlin, Germany  
  - [Abstract](https://meetingcpp.com/2019/Talks/items/Catching_________Unicode_for_Cpp_in_Greater_Detail___2_of_5.html)
  - [Video](https://www.youtube.com/watch?v=FQHofyOgQtM)
  - [Slides](/_presentations/unicode/Meeting%20C++/2019/2019.11.16%20-%20Catching%20⬆️%20-%20Unicode%20for%20C++%20in%20Greater%20Detail%20-%20ThePhD%20-%20Meeting%20C++.pdf)

4. CppCon 2019  
  _Catching ⬆️: The (Baseline) Unicode Plan for C++23_  
  Friday, September 20th, 2019  
  Aurora, Colorado  
  - [Abstract](https://cppcon2019.sched.com/event/7823aebeede8d50e1daa70b5c22ab0a4)
  - [Video](https://www.youtube.com/watch?v=BdUipluIf1E)
  - [Slides](/_presentations/unicode/CppCon/2019/2019.09.20 - Catching ⬆️ - The (Baseline) Unicode Plan for C++23 - ThePhD - CppCon 2019.pdf)
  - [Comments](https://www.reddit.com/r/cpp/comments/de1jy9/cppcon_2019_jeanheyd_meneide_catch_unicode_for_c23/)

5. Study Group 16 - Text and Unicode  
  _A Rudimentary Unicode Abstraction_  
  Wednesday, March 7th, 2018  
  Boston, Massachusetts
  - [Slides](/_presentations/unicode/sg16/2018.03.07/2018.03.07%20-%20ThePhD%20-%20a%20rudimentary%20unicode%20abstraction.pdf)

The current spread of goals is as follows.


### Ⅰ: Core Text Utilities [ 🎉 COMPLETE 🎉 ] 

Finished and documented here:
- [View Types - https://ztdtext.readthedocs.io/en/latest/api.html#views](https://ztdtext.readthedocs.io/en/latest/api.html#views)
- [Encoding Objects - https://ztdtext.readthedocs.io/en/latest/encodings.html](https://ztdtext.readthedocs.io/en/latest/encodings.html)

[ ⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿ ]


### Ⅱ: User Extensibility Hooks for (User) Encodings [ 🎉 COMPLETE 🎉 ]

[Finished and documented here: https://ztdtext.readthedocs.io/en/latest/design/lucky%207%20extensions/speed.html](https://ztdtext.readthedocs.io/en/latest/design/lucky%207%20extensions/speed.html)

[ ⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿ ]


### Ⅲ: Byte Buffers and Streaming [ 🎉 COMPLETE 🎉 ]  

[Finished and Documented here: https://ztdtext.readthedocs.io/en/latest/api/encodings/encoding_scheme.html](https://ztdtext.readthedocs.io/en/latest/api/encodings/encoding_scheme.html)

[ ⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿ ]


### Ⅳ: Normalization Forms [ 5% ]

- All four Unicode Normalization Forms, as specified in [UAX #15](https://unicode.org/reports/tr15/).
  - Canonical Form `nfc`
  - Canonical Form `nfd`
  - Compatibility Form `nfkc`
  - Compatibility Form `nfkd`
- `text_view<Encoding, NormalizationForm, Container>`
- `text<Encoding, NormalizationForm, Container>`

[ ⣿⣀⣀⣀⣀⣀⣀⣀⣀⣀⣀⣀⣀⣀⣀⣀⣀⣀⣀⣀ ]


### Ⅴ: CJK Encoding Tests [ 0% ]

- Implement `gb18030`, the official government Unicode Transformation Format encoding of PRC.
- Implement legacy `shift_jis`/`euc_jp`/`iso2022_jp` legacy encodings.
  - priority goes to `shift_jis`/`euc_jp` as it encodes more traffic.

[ ⣀⣀⣀⣀⣀⣀⣀⣀⣀⣀⣀⣀⣀⣀⣀⣀⣀⣀⣀⣀ ]


### Ⅵ: (_Stretch_) Enhanced Execution Encoding [ 0% ]

- Reach into platform-specific functions to rip out guts of platform's current encoding to ensure preservation of Unicode in:
  - `narrow_execution`
  - `wide_execution`

[ ⣀⣀⣀⣀⣀⣀⣀⣀⣀⣀⣀⣀⣀⣀⣀⣀⣀⣀⣀⣀ ]


### Ⅶ: (_Stretch_) Hyper-Scrutinized Vectorization Implementation [ 0% ]

- Apply vectorization techniques for conversions to pairs of encodings in `ascii`, `utf8`, `utf16`, and `utf32`.

[ ⣀⣀⣀⣀⣀⣀⣀⣀⣀⣀⣀⣀⣀⣀⣀⣀⣀⣀⣀⣀ ]


### Ⅷ: (_Stretch_) C Library for Span-Based Conversions [ 0% ]

- As detailed in proposal [N2440](/_vendor/future_cxx/papers/C%20-%20Efficient%20Character%20Conversions): C functions for fast conversions.
- Cover INCITS/ANSI fees.
- Take functionality through all of WG14, put into C Libraries such as:
  - `musl`.
  - `glibc`.
  - Potentially: new LLVM `libc`.

[ ⣀⣀⣀⣀⣀⣀⣀⣀⣀⣀⣀⣀⣀⣀⣀⣀⣀⣀⣀⣀ ]


### Ⅸ: (_Stretch_) WHATWG Encoding Functionality [ 0% ]

- The WHATWG (https://whatwg.org/) specifies many encodings which are required to support the web.
- Covers developing encodings to handle [encodings covered by the WHATWG Living Standard](https://encoding.spec.whatwg.org/).

[ ⣀⣀⣀⣀⣀⣀⣀⣀⣀⣀⣀⣀⣀⣀⣀⣀⣀⣀⣀⣀ ]


### Ⅹ: (_Stretch_) Strong Exception Guarantee [ 0% ]

- `std::text<Encoding, NormalizationForm, Container>`: strong exception guarantee on all applicable operations.
- `noexcept` container support for `std::text` and `std::text_view`
  - `noexcept` allocator support.
  - Containers operations are made conditionally `noexcept` if possible based on the allocator and movability of the inserted types.

[ ⣀⣀⣀⣀⣀⣀⣀⣀⣀⣀⣀⣀⣀⣀⣀⣀⣀⣀⣀⣀ ]
