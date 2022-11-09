---
layout: post
title: "Cuneicode, and the Future of Text in C"
permalink: /cuneicode-and-the-future-of-text-in-c
feature-img: "/assets/img/2022/11/suit-header.png"
thumbnail: "/assets/img/2022/11/suit-header.png"
draft: true
tags: [C, Standard, String, Text, Unicode, API Design]
excerpt_separator: <!--more-->
---

Following up from the last post, there is a lot more we need to cover. First, due to having to go over the performance implications of everything, but more importantly due to a few folks taking issue with my outright dismissal of the <!--more-->C and C++ APIs (and not showing them in the last post's teaser benchmarks).

Part of this post will be adding to the table from [Part 1](/the-c-c++-rust-string-text-encoding-api-landscape), talking about *why* the API is trash (rather than just taking it for granted that industry professionals, hobbyists, and academic experts have already discovered how trash it is), and unfortunately adding it to the benchmarks we are doing (which means using it in anger). As a refresher, here's where the table we created left off, with all of what we discovered (including errata from comments people sent in):

| Feature Set 👇 vs. Library 👉 | ICU | libiconv | simdutf | encoding_rs/encoding_c | ztd.text |
| Handles Legacy Encodings | ✅ | ✅  | ❌ | ✅ | ✅ |
| Handles UTF Encodings | ✅ | ✅  | ✅ | 🤨 | ✅ |
| Bounded and Safe Conversion API | ✅ | ✅ | ❌ | ✅ | ✅ |
| Assumed Valid Conversion API | ❌ | ❌ | ✅ | ❌ | ✅ |
| Unbounded Conversion API | ❌ | ❌ | ✅ | ❌ | ✅ |
| Counting API | ✅ | ❌ | ✅ | ✅ | ✅ |
| Validation API | ✅ | ❌ | ✅ | ❌ | ✅ |
| Extensible to (Runtime) User Encodings | ❌ | ❌ | ❌ | ❌ | ✅ |
| Bulk Conversions  | ✅ | ✅ | ✅ | ✅ | ✅ |
| Single Conversions | ✅ | ❌ | ❌ | ❌ | ✅ |
| Custom Error Handling | ✅ | 🤨 | 🤨 | ✅ | ✅ |
| Updates Input Range (How Much Read™) | ✅ | ✅ | 🤨 | ✅ | ✅ |
| Updates Output Range (How Much Written™) | ✅ | ✅ | ❌ | ✅ | ✅ |

| Feature Set 👇 vs. Library 👉 | boost.text | utf8cpp | Standard C | Standard C++ | Windows API |
| Handles Legacy Encodings | ❌ | ❌ | 🤨 | 🤨 | ✅ |
| Handles UTF Encodings | ✅ | ✅ | 🤨 | 🤨 | ✅ |
| Bounded and Safe Conversion API | ❌ | ❌ | 🤨 | ✅ | ✅ |
| Assumed Valid Conversion API | ✅ | ✅ | ❌ | ❌ | ❌ |
| Unbounded Conversion API | ✅ | ✅ | ❌ | ❌ | ✅ |
| Counting API | ❌ | 🤨 | ❌ | ❌ | ✅ |
| Validation API | ❌ | 🤨 | ❌ | ❌ | ❌ |
| Extensible to (Runtime) User Encodings | ❌ | ❌ | ❌ | ✅ | ❌ |
| Bulk Conversions  | ✅ | ✅ | 🤨 | 🤨 | ✅ |
| Single Conversions | ✅ | ✅ | ✅ | ✅ | ❌ |
| Custom Error Handling | ❌ | ✅ | ✅ | ✅ | ❌ |
| Updates Input Range (How Much Read™) | ✅ | ❌ | ✅ | ✅ | ❌ |
| Updates Output Range (How Much Written™) | ✅ | ✅ | ✅ | ✅ | ❌ |

In this article, what we're going to be doing is sizing up particularly the standard C and C++ interfaces, benchmarking all of the APIs in the table, and discussing in particular the various quirks and tradeoffs that come with doing things in this manner. We will also be showing off the C-based API that we have spent all this time leading up to, its own tradeoffs, and if it can tick all of the boxes like ztd.text does. The name of the C API is going to be Cuneicode, a portmanteau of Cuneiform (one of the first writing systems) and Unicode (of Unicode Consortium fame).

| Feature Set 👇 vs. Library 👉 | Standard C | Standard C++ | ztd.text | ztd.cuneicode |
| Handles Legacy Encodings | 🤨 | 🤨 | ✅ | ❓ |
| Handles UTF Encodings | 🤨 | 🤨 | ✅ | ❓ |
| Bounded and Safe Conversion API | 🤨 | ✅ | ✅ | ❓ |
| Assumed Valid Conversion API | ❌ | ❌ | ✅ | ❓ |
| Unbounded Conversion API | ❌ | ❌ | ✅ | ❓ |
| Counting API | ❌ | ❌ | ✅ | ❓ |
| Validation API | ❌ | ❌ | ✅ | ❓ |
| Extensible to (Runtime) User Encodings | ❌ | ✅ | ✅ | ❓ |
| Bulk Conversions  | 🤨 | ✅ | ✅ | ❓ |
| Single Conversions | ✅ | ✅ | ✅ | ❓ |
| Custom Error Handling | ✅ | ✅ | ✅ | ❓ |
| Updates Input Range (How Much Read™) | ✅ | ✅ | ✅ | ❓ |
| Updates Output Range (How Much Written™) | ✅ | ✅ | ✅ | ❓ |

First, we are going to thoroughly review why the C API is a failure API, and all the ways it precipitates the failures of the encoding conversions it was meant to cover (including the existing-at-the-time Big5-HKSCS case that it does not support).

Then, we will discuss the C++-specific APIs that exist outside of the C standard. This will include going from `std::wstring_convert`'s deprecated API to the API that sits underneath it to power the string conversions that it provides, `std::codecvt<ExternCharType, InternCharType, StateObject>` and the various derived classes `std::codecvt(_utf8/_utf16/_utf8_utf16)`. We will also talk about how the C API's most pertinent failure leaks into the C++ API, and how that pitfall is the primary reason why Windows cannot properly support UTF-16 in its core C++ standard library offerings.

Finally, we will get to the benchmarks we talked about before, show where ztd.text ztd.cuneicode are in the benchmarks, and then get to the heart of the issue with text APIs in C and C++.




# Standard C

Standard C's primary deficiency is its constant clinging to and dependency upon the "multibyte" encoding and the "wide character" encoding. In the upcoming C23 draft, these have been clarified to be the Literal Encoding (for `"foo"` strings, at compile-time ("translation time")), the *Wide Literal Encoding* (for `L"foo"` strings, at compile-time), *Execution Encoding* (for any `const char*`/`[const char*, size_t]` that goes into run time ("execution time") function calls), and *Wide Execution Encoding* (for any `const wchar_t*`/`[const wchar_t*, size_t]` that goes into run time function calls). In particularly, C relies on the Execution Encoding in in order to go-between UTF-8, UTF-16, UTF-32 or Wide Execution encodings. This is clear from the functions present in both the `<wchar.h>` headers and the `<uchar.h>` headers:

```cpp
// From Execution Encoding to Unicode
size_t mbrtoc32(char32_t* restrict pc32, const char* restrict s,
	size_t n, mbstate_t* restrict ps );
size_t mbrtoc16(char16_t* restrict pc16, const char* restrict s,
	size_t n, mbstate_t* restrict ps );
size_t mbrtoc8(char8_t* restrict pc8, const char* restrict s,
	size_t n, mbstate_t* restrict ps );

// To Execution Encoding from Unicode
size_t c32rtomb(char* restrict s, char32_t c32,
	mbstate_t* restrict ps);
size_t c16rtomb(char* restrict s, char16_t c16,
	mbstate_t* restrict ps);
size_t c8rtomb(char* restrict s, char8_t c8,
	mbstate_t* restrict ps);

// From Execution Encoding to Wide Execution Encoding
size_t mbrtowc(wchar_t* restrict pwc, const char* restrict s,
	size_t n, mbstate_t* restrict ps);
// Bulk form of above
size_t mbsrtowcs(wchar_t* restrict dst, const char** restrict src,
	size_t len, mbstate_t* restrict ps);
// From Wide Execution Encoding to Execution Encoding
size_t wcrtomb(char* restrict s, wchar_t wc,
	mbstate_t* restrict ps);
// Bulk form of above
size_t wcsrtombs(char* restrict dst, const wchar_t** restrict src,
	size_t len, mbstate_t* restrict ps);
```

The naming pattern is "(prefix)(`s`?)(`r`)`to`(suffix)(`s`?)", where `s` means "string" (bulk processing), `r`" means "restartable" (takes a state parameter so it a string can be re-processed by itself), and the core `to` which is just to signify that it goes to the prefix-identified encoding to the suffix-identified encoding. `mb` means "multibyte", `wc` means "wide character", and `c8/16/32` are "UTF-8/16/32", respectively.

Those are the only functions available, and with it comes an enormous dirge of problems that go beyond the basic API design nitpicking of libraries like simdutf or encoding_rs/encoding_c. To understand the naming convention, C relies on a . First and foremost, it does not include all the possible pairings of encodings that it already acknowledges it knows about. Secondly, it does not include full bulk transformations (except in the case of going between execution encoding and wide execution encoding). All in all, it's an exceedingly disappointing offering, as shown by the tables below.

For "Single Conversions", what's provided by the C Standard is as follows:

|         | **mb** | **wc** | **c8** | **c16** | **c32** |
|---------|--------|--------|--------|---------|---------|
| **mb**  | ❌     | ✅     | ✅     | ✅     | ✅      |
| **wc**  | ✅     | ❌     | ❌     | ❌     | ❌      |
| **c8**  | ✅     | ❌     | ❌     | ❌     | ❌      |
| **c16** | ✅     | ❌     | ❌     | ❌     | ❌      |
| **c32** | ✅     | ❌     | ❌     | ❌     | ❌      |

For "Bulk Conversions", what's provided by the C Standard is as follows:

|          | **mbs** | **wcs** | **c8s** | **c16s** | **c32s** |
|----------|---------|---------|---------|----------|----------|
| **mbs**  | ❌      | ✅      | ❌      | ❌      | ❌       |
| **wcs**  | ✅      | ❌      | ❌      | ❌      | ❌       |
| **c8s**  | ❌      | ❌      | ❌      | ❌      | ❌       |
| **c16s** | ❌      | ❌      | ❌      | ❌      | ❌       |
| **c32s** | ❌      | ❌      | ❌      | ❌      | ❌       |

As with the other table, the "✅" is for that conversion sequence being supported, and the "❌" is for no support. As you can see from all the "❌" in the above table, we have effectively missed out on a ton of functionality needed to go to and from Unicode encodings. C **only** provides bulk conversion functions for the "mbs"/"wcs" series of functions, meaning you can kiss any SIMD or other bulk-processing optimizations goodbye for just about every other kind of conversion in C's API, including any UTF-8/16/32 conversions. Also note that C23 and C++20/23 had additional burdens to fix:

- `u""` and `U""` string literals did not have to be UTF-16 or UTF-32 encoded;
- `c16` and `c32` did not have to actually mean UTF-16 or UTF-32 for the execution encoding;
- the C and C++ Committee believed that `mb` could be used as the "UTF-8" through the locale, which is why they left `c8rtomb` and `mbrtoc8` out of the picture entirely until it was fixed up.

These are no longer problems thanks to C++'s Study Group 16, with papers from R.M. Fernandes, Tom Honermann, and myself. If you've [read my previous blog posts](/c-the-improvements-june-september-virtual-c-meeting#n2728---char16_t--char32_t-string-literals-shall-be-utf-16-and-utf-32), I went into detail about how the C and C++ implementations could simply define the standard macros `__STDC_UTF_32__` and `__STDC_UTF_16__` to 1. An implementation could give you a big fat middle finger with respect to what encoding was being used by the `mbrtoc16/32` functions, and also not tell you what is in the `u"foo"` and `U"foo"` string literals. This was an allowance that we worked hard to nuke out of existence, preventing yet another escalation of `char16_t`/`char32_t` and friends ending up in the same horrible situation as `wchar_t` where it's entirely platform (and possibly run time) dependent what encoding is used for those functions. As mentioned in the linked article, we were lucky that nobody was doing the "wrong" thing with it and always provided UTF-32 and UTF-16. This made it easy to hammer the behavior in all current and future revisions of C and C++. This, of course, does not answer why `wchar_t` is actually something to fear, and why we didn't want `char16_t` and `char32_t` to become either of those.

So let's talk about why `wchar_t` is literally The Devil.



## C and `wchar_t`

This is a clause that currently haunts C, and is — hopefully — on its way out the door for the C++ Standard in C++23 or C++26. But, the wording in the C Standard is pretty straight forward (emphasis mine):

> wide character
> 
> value representable by an object of type `wchar_t`, **capable of representing any character in the current locale**
>
> — §3.7.3 Definitions, "wide character", [N3054](https://www.open-std.org/jtc1/sc22/wg14/www/docs/n3054.pdf)

This one definition means that if you have an input into `mbrtowc` that needs to output more than one (1) `wchar_t` for the desired output, there is no appreciable way to describe that in the standard C API. This is because there is no reserved return code for `mbrtowc` to describe needing to serialize a second `wchar_t` into the `wchar_t* restrict pwc`. Furthermore, despite being a pointer type, `pwc` expects only a single `wchar_t` to be output into it. Changing the definition in future standards to allow for 2 or more `wchar_t`'s to be written to `pwc` is a recipe for overwriting the stack for code that updates its standard library but does not re-compile the application to use a larger output buffer. Taking this kind of action would ultimately end up breaking applications in horrific ways (A.K.A, an [ABI Break](/binary-banshees-digital-demons-abi-c-c++-help-me-god-please)), so it is fundamentally unfixable in Standard C. This is why encodings like Big5-HKSCS cannot be used in Standard C. Despite libraries advertising support for them like glibc and its associated locale machinery, they return non-standard and unexpected values to cope with inputs that need to write out two UTF-32 code points for a single indivisible unit of input.

This same reasoning applies to distributions that attempt to use UTF-16 as their `wchar_t` encoding. Similarly to how Big5-HKSCS requires two (2) UTF-32 code points for some inputs, large swaths of the UTF-16 encoded characters use a double code unit sequence. Despite platforms like e.g. Microsoft Windows having the demonstrated **ability** to produce `wchar_t` strings that are UTF-16 based, the standard library must explicitly be UCS-2. If a string is input that requires 2 UTF-16 code units — a leading surrogate pair and its trailing counterpart surrogate code unit — there is no expressible way to do this in C. All that happens is that, even if they recognize an input sequence that generates 2 UTF-16 `wchar_t` code units, it will get chopped in half and the computer library reports "yep, all good". Microsoft is far from the only company in this boat: IBM cannot fix many of their machines, same as Oracle and a slew of others that are ABI-locked into UTF-16 because they tried to adopt Unicode 1.0 faster than everyone else and got screwed over by the Unicode Consortium's crappy UCS-2 version 1.0 design.

This, of course, does not even address that `wchar_t` on specific platforms does not have to be either UTF-16 or UTF-32, and thanks to some weasel-wording in the C Standard and its popularly-endorsed interpretation in compilers like Clang and amongst the C Standards Committee, it can be impossible to detect if even your string literals are appropriately UTF-32, let alone UTF-16. For example, `__STDC_ISO_10646__` can actually be turned off by a compiler because, as far as the C Committee is concerned, it is a reflection of a **run time** property (whether or not `mbrtowc` can handle UTF-32), which is decided by locale (yes, `wchar_t` can depend on locale, like it does on IBM and *BSD-based machines). This makes `__STDC_ISO_10646__` a reflection of a run time property, but something that has to be defined **before**-hand, at compile time, by the compiler. So, the easiest answer — even if your compiler knows it encodes `L"foo"` strings as 32-bit `wchar_t` with UTF-32 code points — is to just advertise that as `0`. It's 100% technically correct to do so,

and 100% technically the worst possible interpretation and conclusion the C Standards Committee could come to.

So, not only is `wchar_t` **not** guaranteed to be Unicode, it is also an opaque bag of nonsense that may or may not be capable of handling the text you encode into it or decode from it! And things just continue to get worse from here.



## C and "multibyte" `const char*` Encodings

As mentioned briefly before, the C and C++ Committee believed that the Execution Encoding could just simply be made to be UTF-8. This was back when people still had faith in locales (an attitude still depressingly available in today's ecosystem, but in a much more damaging and sinister form to be talked about later). In particular, there are no Unicode conversions except those that go through the opaque, implementation-defined Execution Encoding. For example, if you wanted to go from the Wide Execution Encoding (`const wchar*`) to UTF-8, you cannot simply convert directly from a `const wchar_t* wide_str` string — whatever encoding it may be — to UTF-8. You have to:

- set up an intermediate `const char temp[MB_MAX_LEN];` temporary holder;
- call `wcrtomb(temp, *wide_str, …)`;
- feed the data from `temp` into `mbrtoc8(temp, ...)`; and
- loop over the `wide_str` string until you are out of input and intermediate input.

The first and most glaring problem is: what happens if the Execution Encoding is not Unicode? It's an unfortunately frightfully common case, and as much as the Linux playboys love to shill for their platform and the "everything is UTF-8 by default" perspective, they're not being honest with you or really anyone else on the globe. For example, on a freshly installed WSL Ubuntu LTS, with `sudo apt-get update` and `sudo apt-get dist-upgrade` freshly run, when I write a C or C++ program to query what the locale is with `getlocale` and compile that program with Clang 15 with as advanced of a glibc/libstdc++ as I can get, this is what the printout reads:

```sh
=== Encoding Names ===
Literal Encoding: UTF-8
Wide Literal Encoding: UTF-32
Execution Encoding: ANSI_X3.4-1968
Wide Execution Encoding: UTF-32
```

If you look up what "ANSI_X3.4-1968" means, you'll find that it's the most obnoxious and fancy way to spell a particularly old encoding. That is to say, my default locale when I ask and use it in C or C++ — on my brand new Ubuntu 20.04 Focal LTS server, achieved from just pressing "ok" to all the setup options, installing build essentials, and then going out of my way to get the most advanced Clang I can and combine it with the most up-to-date glibc and libstdc++ I can —

is ASCII. 

Not UTF-8. Not Latin-1!



## Just ASCII.

Post-locale, "`const char*` is always UTF-8", "UTF-8 is the only encoding you'll need" world, eh? 🙄

Windows fares no better, pulling out the generic default locale associated with my typical location since the computer's install. This means that if I decide today is a good day to transcode between UTF-16 and UTF-8 the "standard" way, everything that is not either ASCII or Latin-1 will simply be mangled, errored on, or destroyed. I have long since had to teach not only myself, but others how to escape the non-UTF-8 hell on Windows machines. For example, someone sent me a screenshot of a bit of code whose comments looked very much like it was mojibake'd over Telegram:

![](/assets/img/2022/11/telegram-bad-data.png)

Visual Studio was, of course, doing typical Microsoft-text-editor-based Visual Studio things here. It was clear what went down, so I gave them some advice:

![](/assets/img/2022/11/telegram-advice.jpg)

And, 'lo and behold:

![](/assets/img/2022/11/telegram-bingo.png)

Of course, some questions arise. One might be "How did you know it was Windows 1251?". The answer is that I spent a little bit of time in The Mines™ using alternative locales on my Windows machine sometime — Japanese, Chinese, and Russian — and got to experience first-hand how things got pretty messed up by programs. And that's just the tip of the iceberg: Windows 1251 is the most persistent encoding for Cyrillic data into/out of Russia. There's literally an entire Wiki that contains common text sequences and their Mojibake forms when incorrectly interpreted as other forms of encodings, and most Cryllic users are so used to being fucked over by computing in general that they memorized the various UTF and locale-based mojibake results, able to read pure mangled text and translate that in-real-time to what it actually means in their head. (I can't do that: I have to look stuff up.)

There are other ways to transition your application to UTF-8 on Windows, even if you might receive Windows 1251 data or something else. Some folks achieve it by drilling Application Manifests into their executables. But that only works for applications; ztd.text and ztd.cuneicode are libraries. How the hell am I supposed to Unicode-poison an application that consumes my library? The real answer is that there is still no actual solution, and so I spend my time telling others about this crappy world when C and C++ programs inevitably destroy people's data. But, there is one Nuclear Option you can deploy as a Windows user, just to get UTF-8 by-default as the default codepage for C and C++ applications:

![](/assets/img/2022/11/windows-unicode-option.jpg)

Yep, the option to turn on UTF-8 by default is buried *underneath* the new Settings screen, under the "additional clocks" Legacy Settings window on the first tab, into the "Region" Legacy Settings window on the **second** tab ("Administrative"), and then you need to click the "Change system locale" button, check a box, and reset your computer.

But sure, after you do all of that, you get to live in a post-locale world. 🙃



## Post-Locale, or Post-Sanity?

Really, we have not made a post-locale world. The reality of the matter is that there's no clear and standard way to convert from locale-specific data that lurks on so many machines, in so many government/corporate databases, in so many other places. As long as those C APIs are not augmented with clear, direct, non-fuzzy conversions from the data that exists already on people's machines, in people's governments, in people's databases and being interchanged "opaquely" (that's the repeated lie Linux people love to tell me every time I bring this up, that string handling is "opaque" on their platforms), then we will continue to make normal human being's life miserable. Truly, I wish I was kidding: one of the people I am working with on my fancy academic paper that will eventually go into the Journal of Open Source Software (JOSS) is a Frenchman. He has an umlaut (ë) in his name. He asked for a passport from the French government with his parent-given, God-blessed birth name. What he got back was a passport, instead, was this:

> JoÃ«l

No amount of calling in could fix it. The system was broken, because the default locale for French systems handling the data was Latin-1, and serializing the UTF-8 bytes of his name over the web and application forms before mangling them with C-based software and standard libraries that processed government requests. This resulted in the visual bastardization that was sent back to him in shiny, golden, embossed letters. No amount of petitioning could fix it, and how could it? None of the people he could ever get in contact with over the phone or by letter had the technical know-how to understand how deeply broken the C ecosystem was, and so when it was time to renew his passport JoÃ«l didn't bother asking for the umlaut anymore. Instead, JoÃ«l willing transformed himself in the eyes of the government from 🇫🇷Joël🇫🇷 down into bland, ASCII, culture-erased 🇺🇸Joel🇺🇸, just to make the computer systems serving many governments and international transportation providers happy.

Now, for Joël, it's just a funny — and sometimes frustrating — story. After all, being downgraded to Joel isn't so bad when many parts of the internet and even his own government cannot get it right. And sure, the "random" selection for stops/searches when doing international travel seemed to mysteriously drop in frequency now because the passport and boarding pass and plane ticket all match and don't look like a botched fake credential job. But it gets more frustrating. From many states in the U.S. not allowing anything beyond ASCII in their names for both technical (for as much as "C fucking sucks and locales are hot garbage" counts as a technical reason) and deeply bigoted non-technical reasons, to Polish people traveling and getting subjected to an extra stop because their name contains non-ASCII and the plane ticket printer just couldn't handle it so it printed some garbage that did not match their passport: everyone whose name isn't ASCII-clean has suffered at the hands of computer systems purported to make life easier and better. Germany has computer systems whose databases still can't handle a goddamn umlaut, for crying out loud:

[![](/assets/img/2022/11/umlaut-tweet.jpeg)](https://www.youtube.com/watch?v=U6xSmkdz2Nk&t=421s)

And don't forget IEE — you know, the **international** Institute of Electrical and Electronics Engineering under ISO — not handling normal people's names:

[![](/assets/img/2022/11/cl%C3%A9ment-tweet.jpeg)](https://www.youtube.com/watch?v=U6xSmkdz2Nk&t=383s)

C continues to deliver on the promise of making sure every single person whose culture isn't properly crushed into 7-bit cleanliness gets the short end of the stick.



## And It Gets Worse

Because of course it gets worse. The functions I listed previously all have an `r` in the middle of their names; this is an indicator that these functions take an `mbstate_t*` parameter. This means that the state used for the conversion sequence is not taken from its *alternative* location. Which may or may not be a thread-safe, static storage duration `mbstate_t` object maintained by the implementation. It may be `thread_local`, it may not, but whether or not it is thread safe there is still the distinct horribleness that it is an opaquely shared object. That means if, at any point, someone uses the non-`r` versions of the above standard C functions, any functions downstream of them have their state changed out from underneath them. This seems like scripting-language style behavior, where everything is connected to everything else is a jumble of hidden and/or global state, grabbing variables and functionality from wherever and just loading it up with the behavior that happens to be going at the time.

Of course, even using the `r` functions still leaves the need to go through the multibyte character set. Even if you pass in your own `mbstate_t` object, you still have to consult with the (global) locale, and if ay any point someone calls `setlocale("fck.U");` you become liable to deal with that change in further downstream function calls. Helpfully, the C standard manifests this as unspecified behavior, if one function call starts in one locale with one associated encoding, but ends up in another locale with a different associated encoding. Most likely you end up with either hard crashes or strange, undecipherable output even for legal input sequences. All in all, just one footgun after another when it comes to using Standard C in any remotely scalable fashion. It is not surprise that the advice for these functions about their use is "DO. NOT.", which really inspires confidence in the base that every serious computation engine in the world builds on.

"Just ignore it, bro." What is the C Standard, an early 2000s troll? Are we still handing that advice out after all we learned about how it's impossible to "just ignore" this kind of stuff, and that the longer we ignore it the more malignantly brutal it becomes for everybody involved? No? We haven't learned anything yet? Just "work around it"?

Mmmkay!




# Standard C++

When I originally discussed Standard C++, I approached it from its flagship API — `std::wstring_convert<…>` — and all the problems therein. But, there was a layer beneath that I had filed away as "trash", but that could still be used to get around many of `std::wstring_convert<…>`'s glaring issue. For example, `wstring_convert::to_bytes` always returns a new `std::string`-alike by-value, meaning that there's no room to pre-allocate or pre-reserve data (giving it the worst of the allocation penalty and any pessimistic growth sizing as the string is converted). It also always assumes that the "output" type is `char`-based, while the input type is `Elem`-based. Coupled with the by-value, allocated return types, it makes it impossible to save on space or time, or make it interoperable with a wide variety of containers (e.g., `TArray<…>` from Unreal Engine or `boost::static_vector`), requiring an additional copy to but it into something as simple as a `std::vector`.

But, it would be unfair to judge the higher-level — if trashy — convenience API when there is a lower-level one present in virtual-based `codecvt` classes. These are member functions, and so the public-facing API and the protected, derive-ready API are both shown below:

```cpp
template <typename InternalCharType,
	typename ExternalCharType,
	typename StateType>
class codecvt {
public:
	std::codecvt_base::result out( StateType& state,
		const InternalCharType* from,
		const InternalCharType* from_end,
		const InternalCharType*& from_next,
		ExternalCharType* to,
		ExternalCharType* to_end,
		ExternalCharType*& to_next ) const;

	std::codecvt_base::result in( StateType& state,
		const ExternalCharType* from,
		const ExternalCharType* from_end,
		const ExternalCharType*& from_next,
		InternalCharType* to,
		InternalCharType* to_end,
		InternalCharType*& to_next ) const;

	// …

protected:
	virtual std::codecvt_base::result do_out( StateType& state,
		const InternalCharType* from,
		const InternalCharType* from_end,
		const InternalCharType*& from_next,
		ExternalCharType* to,
		ExternalCharType* to_end,
		ExternalCharType*& to_next ) const;

	virtual std::codecvt_base::result do_in( StateType& state,
		const ExternalCharType* from,
		const ExternalCharType* from_end,
		const ExternalCharType*& from_next,
		InternalCharType* to,
		InternalCharType* to_end,
		InternalCharType*& to_next ) const;

	// …
}
```

Now, this template is not supposed to be anything and everything, which is why it additionally has virtual functions on it. And, despite the poorness of the `std::wstring_convert<…>` APIs, we can immediately see the enormous benefits of the API here, even if it is a little verbose:

- it cares about having both a beginning and an end;
- it contains a third pointer-by-reference (rather than using a double-pointer) to allow someone to know where it stopped in its conversion sequences; and,
- it takes a `StateType`, allowing it to work over a wide variety of potential encodings.

This is an impressive and refreshing departure from the usual dreck, as far as the API is concerned. As an early adopter of `codecvt` and `wstring_convert`, however, I absolutely suffered its suboptimal API and the poor implementations, from Microsoft missing `wchar_t` specializations that caused `wstring_convert` to break until they fixed it, or MinGW's patched library deciding today was a good day to always swap the bytes of the input string to produce Big Endian data no matter what parameters were used, it was always a slog and a struggle to get the API to do what it was supposed to do.

But the thought was there. You can see how this API could be the one that delivered C++ out of the "this is garbage nonsense" mines, and delivered all of C++ to the promised land over C++. They even had classes prepared to do just that, committing to UTF-8, UTF-16, and UTF-32 while C was still struggling to get past "`char*` is always (sometimes) UTF-8, just use the multibyte encoding for that":

```cpp
template <typename Elem,
	unsigned long Maxcode = 0x10ffff,
	std::codecvt_mode Mode = (std::codecvt_mode)0
> class codecvt_utf8 : public std::codecvt<Elem, char, std::mbstate_t>;

template <typename Elem,
	unsigned long Maxcode = 0x10ffff,
	std::codecvt_mode Mode = (std::codecvt_mode)0
> class codecvt_utf16 : public std::codecvt<Elem, char, std::mbstate_t>;

template <typename Elem,
	unsigned long Maxcode = 0x10ffff,
	std::codecvt_mode Mode = (std::codecvt_mode)0
> class codecvt_utf8_utf16 : public std::codecvt<Elem, char, std::mbstate_t>;
```

UTF-32 is supported by passing in `char32_t` as the `Elemn` element type. The `codecvt` API was byte-oriented, meaning it was made for serialization. That meant it would do little-endian or big-endian serialization by default, and you had to pass in `std::codecvt_mode::little_endian` to get it to behave. Similarly, it sometimes would generate or consume byte order markers if you passed in `std::codecvt_mode::consume_header` or `std::codecvt_mode::generate_header` (but it **only** generates a header for UTF-16 or UTF-8, NOT for UTF-32 since UTF-32 was considered the "internal" character type for these and therefore not on the "serialization" side, which is what the "external" character type was designated for). It was a real shame that the implementations were fairly lackluster when it first came out because this sounds like (almost) everything you could want. By virtue of being a `virtual`-based interface, you could also add your own encodings to this, which therefore made it both compile-time and run-time extensible by virtue of the design. There was just, unfortunately, one problem...



## The "1:N" Rule

Remember how this interface is tied to the idea of "internal" and "external" characters, and the normal "wide string" versus the internal "byte string"? This is where something sinister leaks into the API, by way of a condition imposed by the C++ standard. Despite managing to free itself from `wchar_t` issues, it reintroduces them by applying a new restriction focused exclusively on `basic_filebuf`-related containers.

> A `codecvt` facet that is used by `basic_­filebuf` ([[file.streams]](https://eel.is/c++draft/file.streams)) shall have the property that if
>
> `do_out(state, from, from_end, from_next, to, to_end, to_next)`
>
> would return `ok`, where `from != from_­end`, then
> 
> `do_out(state, from, from + 1, from_next, to, to_end, to_next)`
>
> shall also return `ok`, and that if
>
> `do_in(state, from, from_end, from_next, to, to_end, to_next)`
>
> would return `ok`, where `to != to_­end`, then
>
> `do_in(state, from, from_end, from_next, to, to + 1, to_next)`
>
> shall also return `ok`.<sup>252</sup>
> 
> — Draft C++ Standard, [§30.4.2.5.3 [locale.codecvt.virtuals] ¶4](https://eel.is/c++draft/locale.codecvt#virtuals-4)

And the footnote reads:

> <sup>252)</sup> Informally, this means that `basic_­filebuf` assumes that the mappings from internal to external characters is 1 to N: that a `codecvt` facet that is used by `basic_­filebuf` can translate characters one internal character at a time.
> 
> — Draft C++ Standard, [§30.4.2.5.3 [locale.codecvt.virtuals] ¶4 Footnote 252](https://eel.is/c++draft/locale.codecvt#footnote-252)

In reality, what this means is that, when dealing with `basic_filebuf` as the thing sitting on top of the `do_in`/`do_out` conversions, you must be able to not only convert 1 element at a time, but also avoid returning `partial_conv` and just say "hey, chief, everything looks `ok` to me!". This means that if someone, say, hands you an incomplete stream from inside the file, you're supposed to be able to read only 1 byte of a 4-byte UTF-8 character, say "hey, this is a good, complete character — `return std::codecvt_mode::ok`!!", and then let the file proceed even if it never provides you any other data.

It's hard to find a proper reason for why this is allowed. If you are always allowed to feed in exactly 1 internal character, and it is always expected to form a complete "N" external/output characters, then insofar as `basic_filebuf` is concerned it's allowed to complete mangle your data. Were you writing to a file? Well, good luck with that. Network share? Ain't that a shame! Manage to open a file descriptor for a pipe and it's wrapped in a `basic_filebuf`? Sucks to be you! Everything good about the C++ APIs gets flushed right down the toilet, all because they wanted to support the — ostensibly weird-as-hell — requirement that you can read things exactly 1 character at a time. Of course, they are not really supporting anything, because in order to avoid having to cache the return value from `in` or `out` on any derived `std::codecvt`-derived class, it just means you can be fed a completely bogus stream and it's just considered… okay. That's it. Nothing you or anyone else can do about such a situation: you get nothing but suffering on your plate for this one.

An increasingly nonsensical part of how this specification works is that there's no real way for the `std::codecvt` class to know that it's being called up-stream by a `std::basic_filebuf`, so either every derived `std::codecvt` object has to be okay with artificial truncation, or the developers of the `std::basic_filebuf` have to simply throw away error codes they are not interested in and ignore any incomplete / broken sequences. It seems like most standard libraries choose the latter, which results in, effectively, all encoding procedures for all files to be broken in the same way `wchar_t` is broken in C, but for literally every encoding type regardless of whether `wchar_t` is in the mix or not.

Even more bizarrely, because of the failures of the specification, `std::codecvt_utf16`/`std::codecvt_utf8` are, in general, meant to handle UCS-2 and not UTF-16[^UCS-2]. (UCS-2 is the Unicode Specification v1.0 "wide character" set that handles 65535 code points maximum, which Unicode has already surpassed quite some time ago.) Nevertheless, most (all?) implementations seem to defy the standard, further making the class's stated purpose in code a lot more worthless. There are also additional quirks that trigger undefined behavior when using this type for text-based or binary-based file writing. For example, under the deprecated `<codecvt>` description for `codecvt_utf16`, the standard says in a bullet point "The multibyte sequences may be written only as a binary file. Attempting to write to a text file produces undefined behavior.".

Fascinating.

If it was not for all of these truly whack-a-doodle requirements, we would likely have no problems. But it's too late: any API that uses virtual functions are calcified for **eternity**. Their interfaces and guarantees can never be changed, because changing and their dependents means breaking very strong binary guarantees made about usage and expectations. I was truly excited to see `std::codecvt`'s interface surpassed its menial `std::wstring_convert` counterpart in ways that actually made it a genuinely forward-thinking API, but it ultimately ends up going in the trash like every other Standard API out there. So close,

yet so far.

The rest of the API is the usual lack of thought put into an API to optimize for speed cases. No way to pass `nullptr` as a marker to the `to/from_end` pointers to say "I genuinely don't care, write like the wind", though on certain standard library implementations you could probably just get away with it (at least until subtraction is done with the `to/from_end` pointers to do things like check sizes). There's also no way to just pass in `nullptr` for the entire `to_*` sets of pointers to say "I just want you to give me the count back"; and indeed, there's no way to compute such a count with the triple-input-pointer, triple-output-pointer API. This is why the libiconv-style of pointer-to-pointer, pointer-to-size API ends up superior: it's able to capture all use cases without presenting problematic internal API choices or external user use choices.

This is, ostensibly, why the `std::wstring_convert` performance and class of APIs suck as well. They ultimately cannot perform a pre-count and then perform a reservation, after doing a basic `from_next - from` check to see if the input is large enough to justify doing a `.reserve(...)`/`.resize()` call before looping and `push_back`/`insert`-ing into the target string using the `.in` and `.out` APIs on `std::codecvt`. You just have to make an over-estimate on the size and pre-reserve, or don't do that and just serialize into a temporary buffer before dumping into the output. (This is the implementation choice that e.g. MSVC makes, doing some ~16 characters at a time before vomiting them into the target string in a loop until `std::codecvt::in/out` exhausts all the input).

There is also another significant problem with the usage of `std::codecvt` for its job; it relies on locale. Spinning up a `std::codecvt` can be expensive due to its interaction with `std::locale` and the necessity of being attached to a locale. It is likely intended that these classes can be used standalone, without attaching it to the locale at all (as their destructors, unlike other locale-based facets) were made public and callable rather than hidden/private. This means they can be declared on the stack and used directly, but there's no apparent direct blessing to allow that to happen.




# But, That Done and Dusts That

C and C++ are now Officially Critiqued™ and hopefully I don't have to have anyone crawl out of the woodwork to talk about X or Y thing again and how I'm not being fair enough[^Imagine-Defense].

Nevertheless, if these APIs are garbage, how do we build our own good one? Clearly, if I have all of this evidence and all of these opinions, assuredly I've been able to make a better API? So, let's try to dig in on that. I already figured out the C++ API in ztd.text, so let's cook up ztd.cuneicode, from the ground up, with a good interface.



## The Most Wonderful API

For a C function, we need to have 4 capabilities, as outlined by the table above.

- Single conversions, to transcode one indivisible unit of information at a time. 
- Bulk conversions, to transcode a large buffer as fast as possible (a speed optimization over single conversion with the same properties).
- Validation, to check whether an input is valid and can be converted to the output encoding (or where there an error would occur in the input if some part is invalid).
- Counting, to know how much output is needed (often with an indication of where an error would occur in the input, if the full input can't be counted successfully).

We also know from the ztd.text blog post and design documentation, as well as the analysis from the previous blog post and the above table, that we need to provide specific information for the given capabilities:

- how much input was consumed (even in the case of an error);
- how much output was written (even in the case of an error);
- that "input read" should only include how much was **successfully** read (e.g., stops before the error happens and should not be a partial read);
- that "output written" should only include how much was **successfully** converted (e.g., points to just after the last successful serialization, and not any partial writes); and,
- that we may need additional state associated with a given encoding to handle it properly (any specific "shift sequences" or held-onto state; we'll talk about this more thoroughly when demonstrating the new API).

It turns out that there is already on C API that does most of what we want design-wise, even if its potential was not realized by the people who worked on it and standardized its interface!



## Borrowing Perfection

This library has the perfect interface design and principles with — as is standard with most C APIs — the worst execution. To review, let's take a look at the libiconv conversion interface:

```cpp
size_t iconv(
	iconv_t cd, // any necessary custom information and state
	char ** inbuf, // an input buffer and how far we have progressed
	size_t * inbytesleft, // the size of the input buffer and how much is left
	char ** outbuf, // an output buffer and how far we have progressed
	size_t * outbytesleft); // the size of the output buffer and how much is left
```

As stated in Part 1, while the libiconv library itself will fail to utilize the interface for the purposes we will list just below, we ourselves can adapt it to do these kinds of operations:

- normal output writing (`iconv(cd, inbuf, inbytesleft, outbuf, outbytesleft)`);
- unbounded output writing (`iconv(cd, inbuf, inbytesleft, outbuf, nullptr)`);
- output size counting (`iconv(cd, inbuf, inbytesleft, nullptr, outbytesleft)`); and,
- input validation (`iconv(cd, inbuf, inbytesleft, nullptr, nullptr)`).

Unfortunately, what is missing most from this API is the "single" conversion handling. But, you can always create a bulk conversion by wrapping a one-off conversion, or create a one-off conversion by wrapping a bulk conversion (with severe performance implications either way). We'll add that to the list of things to include when we hijack and re-do this API.

So, at least for a C-style API, we need 2 separate class of functions for one-off and bulk-conversion. In Standard C, they did this by having `mbrtowc` (without an `s` to signify the one-at-a-time conversion nature) and by having `mbsrtowcs` (with an `s` to signify a whole-string conversion). Finally, the last missing piece here is an "assume valid" conversion. We can achieve this by providing a flag or a boolean on the "state"; in the case of `iconv_t cd`, it would be done at the time when the `iconv_t cd` object is generated. For the Standard C APIs, it could be baked into the `mbstate_t` type (though they would likely never, because adding that might change how big the `struct` is, and thus destroy ABI).

With all of this in mind, we can start a conversion effort for all of the fixed conversions. When I say "fixed", I mean conversions from a specific encoding to another, known before we compile. These will be meant to replace the C conversions of the same style such as `mbrtowc` or `c8rtomb`, and fill in the places they never did (including e.g. single vs. bulk conversions). Some of these known encodings will still be linked to runtime based encodings that are based on the locale, but rather than using them and thinking you're going to be traveling through UTF-8 (like with `wcrtomb`), we'll 




# Static Conversion Functions for C

The function names here are going to be kind of gross, but they will be "idiomatic" standard C. We will be using the same established prefixes from the C Standard group of functions, with some slight modifications to the `mb` and `wc` ones to allow for sufficient differentiation from the existing standard ones. Plus, we will be adding a "namespace" (in C, that means just adding a prefix) of `cnc_` (for "Cuneicode"), as well as adding the letter `n` to indicate that these are explicitly the "sized" functions (much like `strncpy` and friends) and that we are not dealing with null terminators at all in this API. Thusly, we end up with functions that look like this:

```cpp
// Single conversion
cnc_XnrtoYn(size_t* p_destination_buffer_len,
	CharY** p_maybe_destination_buffer,
	size_t* p_source_buffer_len,
	CharX** p_source_buffer,
	cnc_mcstate_t* p_state);

// Bulk conversion
cnc_XsnrtoYsn(size_t* p_destination_buffer_len,
	CharY** p_maybe_destination_buffer,
	size_t* p_source_buffer_len,
	CharX** p_source_buffer,
	cnc_mcstate_t* p_state);
```

As shown, the `s` indicates that we are processing as many elements as possible (historically, `s` would stand for `string` here). The tags that replace `X` and `Y` in the function names, and their associated `CharX` and `CharY` types, are:

| Tags | Character Type | Default Associated Encoding |
| `mc` | `char` | Execution (Locale) Encoding |
| `mwc` | `wchar_t` | Wide Execution (Locale) Encoding |
| `c8` | `char8_t`/`unsigned char` | UTF-8 |
| `c16` | `char16_t` | UTF-16 |
| `c32` | `char32_t` | UTF-32 |

The optional encoding suffix is for the left-hand-side (from, `X`) encoding first, before the right-hand side (to, `Y`) encoding. If the encoding is the default associated encoding, then it can be left off. If it may be ambiguous which tag is referring to which optional encoding suffix, both encoding suffixes are provided. The reason we do not use `mb` or `wc` (like pre-existing C functions) is because those prefixes are tainted forever by API and ABI constraints in the C standard to refer to "bullshit multibyte encoding limited by a maximum output of `MB_MAX_LEN`", and "can only ever output 1 value and is insufficient even if it is picked to be UTF-32", respectively. The new name "`mc`" stands for "multi character", and "`mwc`" stands for — you guessed it — "multi wide character", to make it explicitly clear there's multiple values that will be going into and coming out of these functions.

This means that if we want to convert from UTF-8 to UTF-16, bulk, the function to call is `cnc_c8snrtoc16sn(…)`. Similarly, converting from the Wide Execution Encoding to UTF-32 (non-bulk) would be `cnc_mwcnrtoc32n(…)`. There is, however, a caveat: at times, you may not be able to differentiate solely based on the encodings present, rather than the character type. In those cases, particularly for legacy encodings, the naming scheme is extended by adding an additional suffix directly specifying the encoding of one or both of the ends of the conversion. For example, a function that deliberate encodings from [Punycode](https://en.wikipedia.org/wiki/Punycode) ([RFC](https://www.rfc-editor.org/rfc/rfc3492.txt)) to UTF-32 (non-bulk) would be spelled `cnc_mcnrtoc32n_punycode(…)` and use `char` for `CharX` and `char32_t` for `CharY`. A function to convert specifically from SHIFT-JIS to EUC-JP (in bulk) would be spelled `cnc_mcsnrtomcsn_shift_jis_euc_jp(…)` and use `char` for both `CharX` and `CharY`. Furthermore, since people like to use `char` for UTF-8 despite [associated troubles with `char`'s signedness](/ever-closer-c23-improvements#isspace-u8-%F0%9F%92%A3-0), a function converting from UTF-8 to UTF-16 in this style would be `cnc_mcsnrtoc16sn_utf8(…)`. The function that converts the execution encoding to `char`-based UTF-8 is `cnc_mcsnrtomcsn_exec_utf8(…)`.

The names are definitely a mouthful, but it covers all of the names we could need for any given encoding pair for the functions that are named at compile-time and do not go through a system similar to libiconv. Given this naming scheme, we can stamp out all the core functions between the 5 core encodings present on C and C++-based, locale-heavy systems (UTF-8, UTF-16, UTF-32, Execution Encoding, and Wide Execution Encoding), and have room for additional functions using specific names.

Finally, there is the matter of "conversions where the input is assumed good and valid". `cnc_mcstate_t` has a function that will set its assume valid. The proper `= {}` init of `cnc_mcstate_t` keeps it off as normal. But you can set it explicitly using the function, which helps us cover the "input is valid" bit.

Given all of this, we can demonstrate a small usage of the API here:

```cpp
#include <ztd/cuneicode.h>

#include <ztd/idk/size.h>

#include <stdio.h>
#include <stdbool.h>
#include <string.h>

int main() {
	const char32_t input_data[] = U"Bark Bark Bark 🐕‍🦺!";
	char output_data[ztd_c_array_size(input_data) * 4] = {};
	cnc_mcstate_t state                                = {};
	// set the "do UB shit if invalid" bit to true
	cnc_mcstate_set_assume_valid(&state, true);
	const size_t starting_input_size  = ztd_c_string_array_size(input_data);
	size_t input_size                 = starting_input_size;
	const char32_t* input             = input_data;
	const size_t starting_output_size = ztd_c_array_size(output_data);
	size_t output_size                = starting_output_size;
	char* output                      = output_data;
	cnc_mcerror err                   = cnc_c32snrtomcsn_utf8(
	                       &output_size, &output, &input_size, &input, &state);
	const bool has_err          = err != CNC_MCERROR_OK;
	const size_t input_read     = starting_input_size - input_size;
	const size_t output_written = starting_output_size - output_size;
	const char* const conversion_result_title_str
	     = (const char*)(has_err ? u8"Conversion failed... 😭"
	                             : u8"Conversion succeeded 🎉");
	const size_t conversion_result_title_str_size
	     = strlen(conversion_result_title_str);
	// Use fwrite to prevent conversions / locale-sensitive-probing from
	// fprintf family of functions
	fwrite(conversion_result_title_str, sizeof(*conversion_result_title_str),
	     conversion_result_title_str_size, has_err ? stderr : stdout);
	fprintf(has_err ? stderr : stdout,
	     "\n\tRead: %zu %zu-bit elements"
	     "\n\tWrote: %zu %zu-bit elements\n",
	     (size_t)(input_read), (size_t)(sizeof(*input) * CHAR_BIT),
	     (size_t)(output_written), (size_t)(sizeof(*output) * CHAR_BIT));
	fprintf(stdout, "%s Conversion Result:\n", has_err ? "Partial" : "Complete");
	fwrite(output_data, sizeof(*output_data), output_written, stdout);
	// The stream is (possibly) line-buffered, so make sure an extra "\n" is written
	// out; this is actually critical for some forms of stdout/stderr mirrors. They
	// won't show the last line even if you manually call fflush(…) !
	fwrite("\n", sizeof(char), 1, stdout);
	return has_err ? 1 : 0;
}
```

Which (on a terminal that hasn't lost its mind[^lost-mind]) produces the following output:

```sh
Conversion succeeded 🎉:        
	Read: 19 32-bit elements
	Wrote: 27 8-bit elements
Complete Conversion Result:
Bark Bark Bark 🐕‍🦺! 
```

Of course, as specified in this section's title, this only covers the **canonical** conversions:

|         | **mc** | **mwc**| **c8** | **c16** | **c32** |
|---------|--------|--------|--------|---------|---------|
| **mc**  | ✅     | ✅     | ✅     | ✅     | ✅      |
| **mwc** | ✅     | ✅     | ✅     | ✅     | ✅      |
| **c8**  | ✅     | ✅     | ✅     | ✅     | ✅      |
| **c16** | ✅     | ✅     | ✅     | ✅     | ✅      |
| **c32** | ✅     | ✅     | ✅     | ✅     | ✅      |

|          | **mcs** | **mwcs**| **c8s** | **c16s** | **c32s** |
|----------|---------|---------|---------|----------|----------|
| **mcs**  | ✅      | ✅      | ✅      | ✅      | ✅       |
| **mwcs** | ✅      | ✅      | ✅      | ✅      | ✅       |
| **c8s**  | ✅      | ✅      | ✅      | ✅      | ✅       |
| **c16s** | ✅      | ✅      | ✅      | ✅      | ✅       |
| **c32s** | ✅      | ✅      | ✅      | ✅      | ✅       |

Anything else has the special suffix added, but ultimately it is not incredibly satisfactory. After all, part of the wonderful magic of ztd.text and libogonek is the fact that — at compile-time — they could connect 2 encodings together. Now, there's likely not a way to fully connect 2 encodings at compile-time in C without some of the most disgusting macros I would ever write being spawned from the deepest pits of hell. And, I would likely need an extension or two, like Blocks or Statement Expressions, to make it work out nicely so that it could be used everywhere a normal function call/expression is expected.

Nevertheless, not all is lost. I promised an interface that could **automatically** connect 2 disparate encodings, similar to how [ztd::text::transcode(…)](https://ztdtext.readthedocs.io/en/latest/api/conversions/transcode.html) can give to the ability to [convert between a freshly-created Shift-JIS and UTF-8](/any-encoding-ever-ztd-text-unicode-cpp#let’s-implement-shift-jis) without writing that specific conversion routine yourself. This is critical functionality, because it is the step beyond what Rust libraries like encoding_rs offer, and outstrips what libraries like simdutf, utf8cpp, or Standard C could ever offer. If we do it right, it can even outstrip libiconv, where there is a fixed set of encodings defined by the owner of the libiconv implementation that cannot be extended without recompiling the library and building in new routines. ICU includes functionality to connect two disparate encodings, but the library must be modified/recompiled to include new encoding routines, even if they have the `ucnv_ConvertEx` function that takes any 2 disparate encodings and transcodes through `UChar`s (UTF-16). Part of the promise of this article was that we could not only achieve maximum speed, but allow for an infinity of conversions **within C**.

So let's build the C version of all of this.



## General-Purpose Interconnected Conversions Require Genericity

The collection of the cuneicode functions above are all all both strongly-typed and the encoding is known.In most cases (save for the internal execution and wide execution encodings, where things may be a bit ambiguous to an end-user (but not for the standard library vendor)), there is no need for any intermediate conversion steps. They do not need any potential intermediate storage because both ends of the transcoding operation are known. libiconv provides us with a good idea for what the input and output needs to look like, but having a generic pivot is a different matter. ICU and a few other libraries have an explicit pivot source; other libraries (like encoding_rs) want you to coordinate the conversion from the disparate encoding to UTF-8 or UTF-16 and then to the destination encoding yourself (and therefore provide your own UTF-8/16 pivot). Here's how ICU does it in its `ucnv_convertEx` API:

```cpp
U_CAPI void ucnv_convertEx(
	UConverter *targetCnv, UConverter *sourceCnv, // converters describing the encodings
	char **target, const char *targetLimit, // destination
	const char **source, const char *sourceLimit, // source data
	UChar *pivotStart, UChar **pivotSource, UChar **pivotTarget, const UChar *pivotLimit, // pivot
	UBool reset, UBool flush, UErrorCode *pErrorCode); // error code out-parameter
```

The buffers have to be type-erased, which means either providing `void*`, aliasing-capable[^aliasing] `char*`, or aliasing-capable `unsigned char*`. (Aliasing is when a pointer to one type is used to look at the data of a fundamentally different type; only `char` and `unsigned char` can do that, and `std::byte` if C++ is on the table.) After we type-erase the buffers so that we can work on a "byte" level, we then need to develop what ICU calls `UConverter`s. Converters effectively handle converting between their desired representation (e.g., SHIFT-JIS or EUC-KR) and transport to a given neutral middle-ground encoding (such as UTF-32, UTF-16, or UTF-8). In the case of ICU, they convert to `UChar` objects, which are at-least 16-bit sized objects which can hold UTF-16 code units for UTF-16 encoded data. This becomes the Unicode-based anchor through which all communication happens, and why it is named the "pivot".


### Pivoting: Getting from A to B, through C

ICU is not the first library to come up with this. Featured in libiconv, [libogonek](https://github.com/libogonek/ogonek), my own libraries, encoding_rs (in the examples, but not the API itself), and more, libraries have been using this "pivoting" technique for coming up on a decade and a half now. It is effectively the same platonic ideal of "so long as there is a common, universal encoding that can handle the input data, we will make sure there is an encoding route from A to this ideal intermediate, and then go to B through said intermediate". Let's take a look at `ucnv_convertEx` from ICU again:

```cpp
U_CAPI void ucnv_convertEx (UConverter *targetCnv, UConverter *sourceCnv,
	char **target, const char *targetLimit,
	const char **source, const char *sourceLimit,
	UChar *pivotStart, UChar **pivotSource, UChar **pivotTarget, // ❗❗ Here ❗❗, the pivot
	const UChar *pivotLimit,
	UBool reset, UBool flush, UErrorCode *pErrorCode);
```

The pivot is typed as various levels of `UChar` pointers, where `UChar` is a stand-in for a type wide enough to hold 16 bits (like `uint_least16_t`). More specifically, the `UChar`-based pivot buffer is meant to be the place where *UTF-16 intermediate data is stored when there is no direct conversion between two encodings*. The iconv library has the same idea, except it does not expose the pivot buffer to you. Emphasis mine:

> It provides support for the encodings…
>
> … [huuuuge list] …
>
> It can convert from any of these encodings to any other, **through Unicode conversion.**
>
> — [GNU version of libiconv](https://www.gnu.org/software/libiconv/), September 17th, 2022

In fact, depending on what library you use, you can be dealing with a "pivot", "substrate", or "go-between" encoding that usually ends up being one of UTF-8, UTF-16, or UTF-32. Occasionally, non-Unicode pivots can be used as well but they are exceedingly rare as they often do not accommodate characters from **both** sides of the equation, in a way that Unicode does (or gives room to). Still, just because somebody writes a few decades-old libraries and frameworks around it, doesn't necessarily prove that pivots are the most useful technique. So, are pivots actually useful?

When I wrote my [previous article about generic conversions](/any-encoding-ever-ztd-text-unicode-cpp#the-result), we used the concept of a UTF-32-based pivot to convert between UTF-8 and Shift-JIS, without either encoding routine knowing about the other. Of course, because this is not templated, we cannot use the compile-time checking to make sure the `decode_one` of one "Lucky 7" object and the `encode_one` of the other "Lucky 7" object lines up. So, we instead need a system where encodings pairs identify themselves in some way, and then identify that as the pivot point. That is, for this diagram:

![](/assets/img/2022/11/transcoding-path.png)

And make the slight modification that allows for this:

![](/assets/img/2022/11/transcoding-path-better.png)

The "something" is our intermediate, and it will also be used as the pivot. Of course, we still can't know what that pivot will be, so we will once again use a type-erased bucket of information for that. Ultimately, our final API for doing this will look like this:

```cpp
#include <stddef.h>

typedef enum cnc_mcerror {
	CNC_MCERROR_OK = 0,
	CNC_MCERROR_INVALID_SEQUENCE = 1,
	CNC_MCERROR_INCOMPLETE_INPUT = 2,
	CNC_MCERROR_INSUFFICIENT_OUTPUT = 3
} cnc_mcerror;

struct cnc_conversion;
typedef struct cnc_conversion cnc_conversion;

typedef struct cnc_pivot_info {
	size_t bytes_size;
	unsigned char* bytes;
	cnc_mcerror error;
} cnc_pivot_info;

cnc_mcerror cnc_conv_one_pivot(cnc_conversion* conversion,
	size_t* p_output_bytes_size, unsigned char** p_output_bytes,
	size_t* p_input_bytes_size, const unsigned char** p_input_bytes,
	cnc_pivot_info* p_pivot_info);

cnc_mcerror cnc_conv_pivot(cnc_conversion* conversion,
	size_t* p_output_bytes_size, unsigned char** p_output_bytes,
	size_t* p_input_bytes_size, const unsigned char** p_input_bytes,
	cnc_pivot_info* p_pivot_info);
```

The `_one` suffixed function does one-by-one conversions, and the other is for bulk conversions. We can see that the API shape here looks pretty much **exactly** like libiconv, with the extra addition of the `cnc_pivot_info` structure for the ability to control how much space is dedicated to the pivot. If `p_pivot_info->bytes` is a null pointer, or `p_pivot_info` is, itself, a null pointer, then it will just use some implementation-defined, internal buffer for a pivot. From this single function, we can spawn the entire batch of functionality we initially yearned for in libiconv. But, rather than force you to write `nullptr`/`NULL` in the exact-right spot of the `cnc_conv_pivot` function, we instead just provide you everything you need anyways:

```cpp
// cnc_conv bulk variants
cnc_mcerror cnc_conv(cnc_conversion* conversion,
	size_t* p_output_bytes_size, unsigned char** p_output_bytes,
	size_t* p_input_bytes_size, const unsigned char** p_input_bytes);

cnc_mcerror cnc_conv_count_pivot(cnc_conversion* conversion,
	size_t* p_output_bytes_size,
	size_t* p_input_bytes_size, const unsigned char** p_input_bytes,
	cnc_pivot_info* p_pivot_info);
cnc_mcerror cnc_conv_count(cnc_conversion* conversion,
	size_t* p_output_bytes_size,
	size_t* p_input_bytes_size, const unsigned char** p_input_bytes);

bool cnc_conv_is_valid_pivot(cnc_conversion* conversion,
	size_t* p_input_bytes_size, const unsigned char** p_input_bytes,
	cnc_pivot_info* p_pivot_info);
bool cnc_conv_is_valid(cnc_conversion* conversion,
	size_t* p_input_bytes_size, const unsigned char** p_input_bytes);

cnc_mcerror cnc_conv_unbounded_pivot( cnc_conversion* conversion,
	unsigned char** p_output_bytes,
	size_t* p_input_bytes_size, const unsigned char** p_input_bytes,
	cnc_pivot_info* p_pivot_info);
cnc_mcerror cnc_conv_unbounded(cnc_conversion* conversion,
	unsigned char** p_output_bytes,
	size_t* p_input_bytes_size, const unsigned char** p_input_bytes);

// cnc_conv_one single variants
cnc_mcerror cnc_conv_one(cnc_conversion* conversion,
	size_t* p_output_bytes_size, unsigned char** p_output_bytes,
	size_t* p_input_bytes_size, const unsigned char** p_input_bytes);

cnc_mcerror cnc_conv_one_count_pivot(cnc_conversion* conversion,
	size_t* p_output_bytes_size,
	size_t* p_input_bytes_size, const unsigned char** p_input_bytes,
	cnc_pivot_info* p_pivot_info);
cnc_mcerror cnc_conv_one_count(cnc_conversion* conversion,
	size_t* p_output_bytes_size,
	size_t* p_input_bytes_size, const unsigned char** p_input_bytes);

bool cnc_conv_one_is_valid_pivot(cnc_conversion* conversion,
	size_t* p_input_bytes_size, const unsigned char** p_input_bytes,
	cnc_pivot_info* p_pivot_info);
bool cnc_conv_one_is_valid(cnc_conversion* conversion,
	size_t* p_input_bytes_size, const unsigned char** p_input_bytes);

cnc_mcerror cnc_conv_one_unbounded_pivot(cnc_conversion* conversion,
	unsigned char** p_output_bytes, size_t* p_input_bytes_size,
	const unsigned char** p_input_bytes, cnc_pivot_info* p_pivot_info);
cnc_mcerror cnc_conv_one_unbounded(cnc_conversion* conversion,
	unsigned char** p_output_bytes,
	size_t* p_input_bytes_size, const unsigned char** p_input_bytes);
```

It's a lot of declarations, but wouldn't you be surprised that the internal implementation of almost all of these is just one function call!

```cpp
bool cnc_conv_is_valid(cnc_conversion* conversion,
	size_t* p_bytes_in_count,
     const unsigned char** p_bytes_in) {
	cnc_mcerror err = cnc_conv_pivot(conversion,
		NULL, NULL,
		p_bytes_in_count, p_bytes_in,
		NULL); 
	return err == CNC_MCERROR_OK;
}
```

It is mostly for convenience to provide these functions. Since the implementation is so simple it warrants giving people exactly what they want, so they can communicate what they're doing in as normal a way as they can possibly manage to rather than the `NULL`/`nullptr` splattering that does not communicate anything to an external user why exactly someone is doing that with the function. Still, for as much as I talk these functions up, there's two very important bits I've been sort of skirting around:

- How the **heck** do we get a `cnc_conversion*` pointer?
- How do we make sure we provide **generic** connection points between random encodings?

Well, strap in, because we are going to be crafting a reusable, general-purpose encoding library that allows for **run time** extension of the available encodings.




# Cuneicode and the Encoding Registry

As detailed in Part 1 and hinted at above, libiconv — and many other existing encoding infrastructures — do not provide a way to expand their encoding knowledge at run time. They ship with a fixed set of encodings, and you must either directly modify the library or directly edit data files in order to coax more encodings out of the interface. In the case of Standard C, sometimes that means injecting more files into the system locale files, or other terrible things. We need a means of loading up and controlling a central place where we can stuff all our encodings. Not only that, but we also:

- need to allow for controlling all allocations made; and,
- need to allow for loading up an encoding registry with any "defaults" the library may have ready for us.

So, we need to come up with a new entity for the API that we will term a `cnc_conversion_registry`. As the name implies, it's a place to store all of our conversions, and thusly a brief outline of the most important parts of the API should look like this:

```cpp
typedef void*(cnc_allocate_function)(size_t requested_size, size_t alignment,
	size_t* p_actual_size, void* user_data);
typedef void*(cnc_reallocate_function)(void* original, size_t requested_size,
	size_t alignment, size_t* p_actual_size, void* user_data);
typedef void*(cnc_allocation_expand_function)(void* original, size_t original_size,
	size_t alignment, size_t expand_left, size_t expand_right, size_t* p_actual_size,
	void* user_data);
typedef void*(cnc_allocation_shrink_function)(void* original, size_t original_size,
	size_t alignment, size_t reduce_left, size_t reduce_right, size_t* p_actual_size,
	void* user_data);
typedef void(cnc_deallocate_function)(
	void* ptr, size_t ptr_size, size_t alignment, void* user_data);

typedef struct cnc_conversion_heap {
	void* user_data;
	cnc_allocate_function* allocate;
	cnc_reallocate_function* reallocate;
	cnc_allocation_expand_function* shrink;
	cnc_allocation_shrink_function* expand;
	cnc_deallocate_function* deallocate;
} cnc_conversion_heap;

cnc_conversion_heap cnc_create_default_heap(void);

typedef enum cnc_open_error {
	CNC_OPEN_ERROR_OK = 0,
	CNC_OPEN_ERROR_NO_CONVERSION_PATH = -1,
	CNC_OPEN_ERROR_INSUFFICIENT_OUTPUT = -2,
	CNC_OPEN_ERROR_INVALID_PARAMETER = -3,
	CNC_OPEN_ERROR_ALLOCATION_FAILURE = -4
} cnc_open_error;

typedef enum cnc_registry_options {
	CNC_REGISTRY_OPTIONS_NONE = 0,
	CNC_REGISTRY_OPTIONS_EMPTY = 1,
	CNC_REGISTRY_OPTIONS_DEFAULT = CNC_REGISTRY_OPTIONS_NONE,
} cnc_registry_options;

cnc_open_error cnc_open_registry(cnc_conversion_registry** p_out_registry, cnc_conversion_heap* p_heap,
	cnc_registry_options registry_options);
cnc_open_error cnc_new_registry(cnc_conversion_registry** p_out_registry,
	cnc_registry_options registry_options);
```

This is a LOT to digest. So, we're going to walk through it, from top-to-bottom. The first 5 are function type definitions: they define the 5 different core operations an allocator can perform. Following the order of the type definitions:

- `cnc_allocate_function`: an allocation function. Creates/acquires memory to write into. That memory can come from anywhere, so long as it contains as much as size was requested. It can give more space than requested (due to block size or for alignment purposes), and so the function takes the actual size as a pointer parameter to give it back to the end-user.
- `cnc_reallocate_function`: a reallocation function. Takes an already-allocated block of memory and sees if it can potentially expand it in place or move it to another place (perhaps using memory relocation) with a larger size. Might result in a `memcpy` action to get the memory from one place to another place, or might do nothing and simply return `nullptr` while not doing anything to the original pointer. Tends to be used as an optimization, and may perhaps be a superset of the `cnc_allocation_expand_function`.
- `cnc_allocation_expand_function`: an expansion function. This function takes an already-done allocation and attempts to expand it in-place. If it cannot succeed at expanding to the left (before) or right (after) of the memory section by the requested amounts, it will simply return `nullptr` and do nothing. Returns a new pointer by return value and files out the actual size by a `size_t` pointer value.
- `cnc_allocation_shrink_function`: a shrinking function. This function takes an already-done allocation and attempts to shrink it in-place. It if cannot succeed at shrinking from the left (before or right (after) of the memory section by the requested amounts, it will simply return `nullptr` and do nothing. Returns a new pointer by return value and files out the actual size by a `size_t` pointer value.
- `cnc_deallocation_function`: a deallocation function. Releases previously-allocated memory.

From there, we compose a heap that contains one of each of the above functions, plus a `void*` which acts as a user data that goes into the heap. The user data's purpose is to provide any additional information that may be needed contextually by this heap to perform its job (for example, a pointer to an span of memory that is then used as a raw arena). 99% of people will ignore the existence of the heap, however, and just use either `cnc_create_default_heap`, or just call `cnc_new_registry` which will create a defaulted heap for you (that just shills out to `malloc` and friends) and then passes it to `cnc_open_registry`.

Finally, there's the registry options. Occasionally, it's useful to create an entirely empty registry, so there's a `CNC_REGISTRY_OPTIONS_EMPTY` for that, but otherwise the default is to stuff the registry with all of the pre-existing encodings. So, we can create a registry for this by doing:

```cpp
cnc_conversion_registry* registry = NULL;
cnc_open_error reg_err            = cnc_new_registry(&registry, CNC_REGISTRY_OPTIONS_DEFAULT);
```

Which is surprisingly simple, despite all the stuff we talked about. The various error codes come from the `cnc_open_error` enumeration, and the names themselves explain pretty clearly what could happen. Some of the error codes don't matter for this specific function, because it's just opening the registry. The most we could run into is a `CNC_OPEN_ERROR_ALLOCATION_FAILURE` or `CNC_OPEN_ERROR_INVALID_PARAMETER`; otherwise, we will just get `CNC_OPEN_ERROR_OK`! Assuming that we did, in fact, get `CNC_OPEN_ERROR_OK`, we can move on to the next part, which is opening/`new`ing up a `cnc_conversion*` from our freshly created `cnc_conversion_registry`.



## Creating a cuneicode Conversion

Dealing with allocations can be a pretty difficult task. As with the `cnc_new_registry` function, we are going to provide a number of entry points that simply shill out to the heap passed in during registry creation so that, once again, 99% of users do not have to care where their memory comes from for these smaller objects. But, it's still important to let users override such defaults and control the space: this is paramount to allow for a strictly-controlled embedded implementation that can compile and run the API we are presenting here. So, let's get into the (thorny) rules of both creating a conversion object, and providing routines to give our **own** conversion routines. First, let's start with creating a conversion object to use:

```cpp
cnc_open_error cnc_conv_new(cnc_conversion_registry* registry,
	const char* from, const char* to,
	cnc_conversion** out_p_conversion,
	cnc_conversion_info* p_info);

cnc_open_error cnc_conv_new_n(cnc_conversion_registry* registry,
	size_t from_size, const char* from,
	size_t to_size, const char* to,
	cnc_conversion** out_p_conversion,
	cnc_conversion_info* p_info);

cnc_open_error cnc_conv_new_n_select(cnc_conversion_registry* registry,
	size_t from_size, const char* from,
	size_t to_size, const char* to,
	cnc_indirect_selection_function* selection,
	cnc_conversion** out_p_conversion,
	cnc_conversion_info* p_info);

cnc_open_error cnc_conv_open(
	cnc_conversion_registry* registry, const char* from, const char* to,
	cnc_conversion** out_p_conversion, size_t* p_available_space, unsigned char* space,
	cnc_conversion_info* p_info);

cnc_open_error cnc_conv_open_n(cnc_conversion_registry* registry,
	size_t from_size, const char* from,
	size_t to_size, const char* to,
	cnc_conversion** out_p_conversion,
	size_t* p_available_space, void* space,
	cnc_conversion_info* p_info);

cnc_open_error cnc_conv_open_n_select(cnc_conversion_registry* registry,
	size_t from_size, const char* from,
	size_t to_size, const char* to,
	cnc_indirect_selection_function* selection,
	cnc_conversion** out_p_conversion,
	size_t* p_available_space, void* space,
	cnc_conversion_info* p_info);
```

As shown with the registry APIs, there's 2 distinct variants: the `_open` and `_new`. `_new` pulls its memory from the heap passed in during registry creation. However, sometimes that's not local-enough for some folks. Therefore, the `_open` variant of the functions ask for a pointer to a `size_t*` for the amount of space is available, and a `void* space` that contains all the space. Each set of APIs takes a `from` name and a `to` name: these are encoding names that are compared in a specific manner. That is:

- it is basic ASCII Latin Alphabet (A-Z, a-z) case-insensitive;
- ASCII `_`, `-`, `.` (period), and space are ignored;
- and the input must be UTF-8.

The reason that the rules are like this is so `"UTF-8"` and `"utf-8"` and `"utf_8"` and `"Utf-8"` are all considered identical. This is different from Standard C and C++, where `setlocale` and `getlocale` are not required to do any sort of invariant-folding comparison and instead can consider  `"C.UTF-8"`, `"C.Utf-8"`, `"c.utf-8"` and similar name variations as completely different. That is, while one platform will affirm that `"C.UTF-8"` is a valid locale/encoding, another platform will reject this despite having the moral, spiritual, and semantic equivalent of `"C.UTF-8"` because you spelled it with lowercase letters rather than some POSIX-blessed "implementation-defined" nutjobbery. Perhaps in the future I could provide Unicode-based title casing/case folding, but at the moment 99% of encoding names are in mostly-ASCII identifiers. (It could be possible in the future to provide a suite of translated names for the `to` and `from` codes, but that is a bridge we can cross at a later date thankfully.)

The `_n` and non-`_n` functions are just variations on providing a size for the `from` and `to` names; this makes it easy not to require allocation if you parse a name out of another format (e.g., passing in a validated sub-input that identifies the encoding from a buffer that contains an `<?xml … ?>` encoding tag in an XHTML file, or the `<meta>` tag). If you don't call the `_n` functions, we do the C thing and call `strlren` on the input `from` and `to` buffers. It's not great, but momentum is momentum: C programmers and the APIs they use/sit beneath them on their systems expect footgun-y null terminated strings, no matter how many times literally everyone gets it wrong.

Knowing all this, we can talk about *direct* matches and *indirect* matches, which comes up in the `cnc_conversion_info` structure:

```cpp
typedef struct cnc_conversion_info {
	const ztd_char8_t* from_code_data;
	size_t from_code_size;
	const ztd_char8_t* to_code_data;
	size_t to_code_size;
	bool is_indirect;
	const ztd_char8_t* indirect_code_data;
	size_t indirect_code_size;
} cnc_conversion_info;
```

The `(to|from)_code_(data/size)` fields should be self-explanatory: when the conversion from `from` to `to` is found, it hands the user the sized strings of the found conversions. These names should compare equal under the function `ztdc_is_encoding_name_equal_n_c8(…)` to the `from`/`to` code passed in to any of the `cnc_conv_new_*`/`cnc_conv_open_*` functions. Note it may not be identical (even if they are considered equivalent), and so the name provided in the `cnc_conversion_info` structure is what is stored inside of the registry, and not the name provided to the function call.

The interesting bit is the `is_indirect` boolean value and the `indirect_code_(data/size)` fields. If `is_indirect` is true, then the `indirect_` fields will be populated with the name (and the size of the name) of the **indirect encoding that is used as a pivot** between the two encoding pairs!


### Indirect Encoding Connection

If we are going to have a way to connect two entirely disparate encodings through a common medium, then we need to be able to direct an encoding through an intermediate. This is where indirect conversions come in. The core idea is, thankfully, not complex, and works as follows:

- if there is an encoding conversion from "`from`" to "`{Something}`";
- and, if there is an encoding from "`{Something}`" to "`to`";
- then, a conversion entry will be created that internally connects `from` to `to` through `{Something}` as the general-purpose pivot.

So, for a brief second, if we assumed we have an encoding conversion from an encoding called "`SHIFT-JIS`" to "`UTF-32`", and we had an encoding from "`UTF-32`" to "`UTF-8`", we could simply ask to go from "`Shift-JIS`" to "`UTF-8`" **without** explicitly writing that encoding conversion ourselves. Since cuneicode comes with an encoding conversion that does Shift-JIS ➡ UTF-32 and UTF-32 ➡ UTF-8, we can try out the following code ourselves and verify it works with the APIs we have been discussing up until now. This is the exact same example we had [back in the C++ article](/any-encoding-ever-ztd-text-unicode-cpp#whew-alright-does-it-work).

Step one is to open a registry, and then open the conversion pair that we want:

```cpp
#include <ztd/cuneicode.h>

#include <ztd/idk/size.h>

#include <stdio.h>
#include <stdbool.h>

int main() {
	cnc_conversion_registry* registry = NULL;
	{
		cnc_open_error err
		     = cnc_registry_new(&registry, CNC_REGISTRY_OPTIONS_DEFAULT);
		if (err != CNC_OPEN_ERROR_OK) {
			fprintf(stderr, "[error] could not open a new registry.");
			return 2;
		}
	}

	cnc_conversion* conversion          = NULL;
	cnc_conversion_info conversion_info = {};
	{
		cnc_open_error err = cnc_conv_new(
		     registry, "shift-jis", "utf-8", &conversion, &conversion_info);
		if (err != CNC_OPEN_ERROR_OK) {
			fprintf(stderr, "[error] could not open a new registry.");
			cnc_registry_delete(registry);
			return 2;
		}
	}

	// …
```

If we fail, we bail and return `2` out of `main`. But, if not, we can keep going. Note that, by this point, the `conversion_info` variable has been filled in, so now we can use it to get information about what we opened up into the `cnc_conversion*` handle:

```cpp
	// …

	fprintf(stdout, "Opened a conversion from \"");
	fwrite(conversion_info.from_code_data,
	     sizeof(*conversion_info.from_code_data), conversion_info.from_code_size,
	     stdout);
	fprintf(stdout, "\" to \"");
	fwrite(conversion_info.to_code_data, sizeof(*conversion_info.to_code_data),
	     conversion_info.to_code_size, stdout);
	if (conversion_info.is_indirect) {
		fprintf(stdout, "\" (through \"");
		fwrite(conversion_info.indirect_code_data,
		     sizeof(*conversion_info.indirect_code_data),
		     conversion_info.indirect_code_size, stdout);
		fprintf(stdout, "\").");
	}
	else {
		fprintf(stdout, "\".");
	}
	fprintf(stdout, "\n");

	// …
```

Executing the code up until this point, we'll get something like:

> ```sh
> Opened a conversion from "shift-jis" to "utf8" (through "utf32").
> ```

which is what we were expecting. Right now, cuneicode only has a conversion routines between Shift-JIS ⬅➡ UTF-32, so it only has one "indirect" encoding to pick from. The rest of this code should look familiar to the example given above for the compile-time known encoding conversions, save for the fact that we are passing values through `unsigned char*` rather than any strongly-typed `const char*` or `char8_t*` types. That means we need to get the array sizes in bytes (not that it matters too much, since the input and output values are in `char` and `unsigned char` arrays):

```cpp
	// …

	const char input_data[] = "all according to , ufufufu!-5r3z2fqepc";
	unsigned char output_data[ztd_c_array_size(input_data)] = {};

	const size_t starting_input_size  = ztd_c_string_array_size(input_data);
	size_t input_size                 = starting_input_size;
	const unsigned char* input        = (const unsigned char*)&input_data[0];
	const size_t starting_output_size = ztd_c_array_size(output_data);
	size_t output_size                = starting_output_size;
	unsigned char* output             = (unsigned char*)&output_data[0];
	cnc_mcerror err
	     = cnc_conv(conversion, &output_size, &output, &input_size, &input);
	const bool has_err          = err != CNC_MCERROR_OK;
	const size_t input_read     = starting_input_size - input_size;
	const size_t output_written = starting_output_size - output_size;
	const char* const conversion_result_title_str
	     = (const char*)(has_err ? u8"Conversion failed... 😭"
	                             : u8"Conversion succeeded 🎉");
	const size_t conversion_result_title_str_size
	     = strlen(conversion_result_title_str);
	// Use fwrite to prevent conversions / locale-sensitive-probing from
	// fprintf family of functions
	fwrite(conversion_result_title_str, sizeof(*conversion_result_title_str),
	     conversion_result_title_str_size, has_err ? stderr : stdout);
	fprintf(has_err ? stderr : stdout,
	     "\n\tRead: %zu %zu-bit elements"
	     "\n\tWrote: %zu %zu-bit elements\n",
	     (size_t)(input_read), (size_t)(sizeof(*input) * CHAR_BIT),
	     (size_t)(output_written), (size_t)(sizeof(*output) * CHAR_BIT));
	fprintf(stdout, "%s Conversion Result:\n", has_err ? "Partial" : "Complete");
	fwrite(output_data, sizeof(*output_data), output_written, stdout);
	// the stream (may be) line-buffered, so make sure an extra "\n" is written
	// out this is actually critical for some forms of stdout/stderr mirrors; they
	// won't show the last line even if you manually call fflush(…) !
	fprintf(stdout, "\n");

	// clean up resources
	cnc_conv_delete(conversion);
	cnc_registry_delete(registry);
	return has_err ? 1 : 0;
}
```

Running the above portion of the code, we can see the following output:

> ```sh
> Conversion succeeded 🎉
> 	Read: 35 8-bit elements
> 	Wrote: 39 8-bit elements
> Complete Conversion Result:
> all according to けいかく, ufufufu!
> ```

So, there we have it. A general-purpose pivoting mechanism that can choose an intermediate and allow us to pivot through to it! That means we have covered most of what is inside of the table even when we use an encoding that is as obnoxious to write an implementation against such as Punycode. Of course, despite demonstrating it can go through an indirect/intermediate encoding, that does not necessarily prove that we can do that for **any** encoding we want. The algorithm inside of cuneicode preferences conversions to and from UTF-32, UTF-8, and UTF-16 before any other encoding, but after that it's a random grab bag of whichever matching encoding pair is discovered first.

This can, of course, be a problem. You may want to bias the selection of the intermediate encoding one way or another; to solve this problem, we just have to add another function call that takes a filtering/"selecting" function.


### Indirect Control: Choosing an Indirect Encoding

Because this is C, we just add some more prefixes on to the existing collection of function names, so we end up with a variant of `cnc_conv_new` that is instead named `cnc_conv_new_select` and its friends:

```cpp
typedef bool(cnc_indirect_selection_function)(size_t from_size,
     const char* from, size_t to_size,
     const char* to, size_t indirect_size,
     const char* indirect);

cnc_open_error cnc_conv_new_n_select(cnc_conversion_registry* registry,
	size_t from_size, const char* from,
	size_t to_size, const char* to,
	cnc_indirect_selection_function* selection, // ❗ this parameter
	cnc_conversion** out_p_conversion, cnc_conversion_info* p_info);
```

A `cnc_indirect_selection_function` type effectively takes the from name, the to name, and the indirect name and passes them to a function that returns a `bool`. This allows a function to wait for e.g. a specific indirect name to select, or maybe will reject any conversion that features an indirect conversion at all (the indirect name will be a null pointer to signify that it's a direct conversion). For example, here's a function that will only allow Unicode-based go-betweens:

```cpp
#include <ztd/cuneicode.h>

#include <stdbool.h>

bool filter_non_unicode_indirect(size_t from_size, const char* from,
	size_t to_size, const char* to,
	size_t indirect_size, const char* indirect) {
	// unused warnings removal
	(void)from_size;
	(void)from;
	(void)to_size;
	(void)to_size;
	if (indirect == nullptr) {
		// if there's no indirect, then it's a direct conversion
		// which is fine.
		return true;
	}
	return ztdc_is_unicode_encoding_name_n(indirect_size, indirect);
}
```

This function might come in handy to guarantee, for example, that there's a maximum chance that 2 encodings could convert between each other. Typically, Unicode's entire purpose is to enable going from one encoded set of text to another without any loss, whether through publicly available and assigned code points or through usage of the private use area. A user can further shrink this surface area by demanding that the go-between is something like UTF-8, which can come particularly in handy for UTF-EBCDIC which has many bit-level similarities with UTF-8 that can be used for optimization purposes as a go-between.

cuneicode itself, when a version of the `cnc_conv_(open|new)` is used, provides a function that simply just returns `true`. This is because cuneicode, internally, has special mechanisms that directly scans a subset of the list of [known Unicode encodings](https://ztdcuneicode.readthedocs.io/en/latest/known%20unicode%20encodings.html) and checks them first. If there's a conversion routine stored in the registry to or from for UTF-8, UTF-16, and UTF-32, it will select and prioritize those first before going on to let the function pick whatever happens to be the first one. The choice is unspecified and not stable between invocations of the `cnc_conv` creation functions, but that's because I'm reserving the right to improve the storage of the conversion routines in the registry and thus might need to change the data structures and their iteration paths / qualities in the future.

Still, there's something more important than these minute details we've been discussing here. In particular, one of the things hinted at by [Part 1](/the-c-c++-rust-string-text-encoding-api-landscape) was this interface — despite it doing things like updating the input/output pointers as well as the input/output sizes — could be fast. Now that we have both the static conversion sections and the registry for this C library, is it possible to be fast? Can we compete with the Big Dogs™ like ICU and encoding_rs for their conversion routines?




# Faster than Fast: Unicode Conversions at the Edge

Let's cut right to the chase: can all of this messing around with a registry — or the static, known-encoding functions — allow us to reach maximum speed in C? Ideally, we created the registry to allow someone to sub in an implementation for transcoding between two encodings in a way that was the most advantageous for their platform. And it would be easy to see why, once we present the below graphs. But, before we do, let's make sure to contextualize all of the results in the upcoming graphs. All of these graphs have the following properties:

- The latest of each library was used as of 21 October, 2022.
- Windows 10 Pro machine, general user processes running in the background (but machine not being used).
- AMD Ryzen 5 3600 6-Core @ 3600 MHz (12 Logical Processors), 32.0 GB Physical Memory
- Clang 15.0.2, latest available Clang at the time of generation with MSVC ABI.
- Entire software stack for every dependency build under default CMake flags (including ICU and libiconv from vcpkg).
- Anywhere from 150 to 10million samples per iteration, with mean (average) of 100 iterations forming transparent dots on graph.
- Each bar graph is mean of the 100 iterations, with provided standard deviation-based error bars.
- In general, unless explicitly noted, the fastest possible API under the constraints was used to produce the data.
	- "Unbounded" means that, where shown, the available space left for writing was not considered.
	- "Unchecked" means that, where shown, the input was not validated before being converted.
	- "Well-Formed", in the title, means that the input was well-formed (we do not do error benchmarks (yet)).
	- "Assumed Valid", in the title, means that **every** benchmarked method shown in the graph does not perform input validation of any kind.

There are many entries in the graph, most of them derived from the critiques done here nd in [Part 1](/the-c-c++-rust-string-text-encoding-api-landscape). They are:

- [simdutf](https://github.com/simdutf/simdutf)
- [iconv](https://www.gnu.org/software/libiconv/) ([patched vcpkg](https://vcpkg.io/en/packages.html) version, search "libiconv")
- [boost.text](https://github.com/tzlaine/text) (proposed for Boost, in limbo...?)
- [utf8cpp](https://github.com/nemtrif/utfcpp)
- [ICU](https://unicode-org.github.io/icu/userguide/conversion/converters.html) (plain libicu & friends, and presumably not the newly released ICU4X, or ICU4C/ICU4J)
- [encoding_rs](https://github.com/hsivonen/encoding_rs) through it's [encoding_c](https://github.com/hsivonen/encoding_c) binding library
- the Windows API, both [`MultiByteToWideChar`](https://docs.microsoft.com/en-us/windows/win32/api/stringapiset/nf-stringapiset-multibytetowidechar) and [`WideCharToMultiByte`](https://docs.microsoft.com/en-us/windows/win32/api/stringapiset/nf-stringapiset-widechartomultibyte)
- [ztd.text](https://github.com/soasis/text/), the initial C++ version of this library leading [P1629](/_vendor/future_cxx/papers/d1629.html)
- [ztd.cuneicode](https://github.com/soasis/cuneicode), the initial C version of this library which is in-part described by [N3031 and its Revisions](/_vendor/future_cxx/papers/C%20-%20Restartable%20and%20Non-Restartable%20Character%20Functions%20for%20Efficient%20Conversions.html)
- [CTRE](https://compile-time-regular-expressions.readthedocs.io/en/latest/), as requested by the library author for their UTF-8 to UTF-32 iterator-based conversion routine.

There are tons of benchmarks that need to be run, and they do not even cover the full gamut of things that can and should be tested. A full listing of all the graphs can be found on the [ztd.text benchmarking page](https://ztdtext.readthedocs.io/en/latest/benchmarks.html), but some of the more interesting ones will be shown here to talk about performance and other metrics. First up, the teaser graphs from last time (with some updates since last time):



![](/assets/img/2022/11/utf32-to-utf8-well_formed.png)

![](/assets/img/2022/11/utf32-to-utf8-well_formed-assume_valid.png)


















Finally, the `_select` functions off an intriguing intervention that is more relevant when selecting a conversion routine.


















[^UCS-2]: See [[depr.locale.stdcvt](https://eel.is/c++draft/depr.locale.stdcvt)].
[^Imagine-Defense]: Imagine having such a crap API with such shitty preconditions that it can't stand on its own merits and instead needs someone to painstakingly "defend" its honor. Amazing work from people who usually spend half of their time trying to convince me that Technology is Definitely A Meritocracy™ and not subject to any of the usual human forces that literally everything else deals with.
[^lost-mind]: This includes Windows Terminal, a handful of Command Prompt shims, Powershell (most of the time), the Console on Mac OS, and (most) Linux Terminals not designed by people part of the weird anti-Unicode Fiefdoms that exist in the many Canon *nix Universes.
[^aliasing]: Aliasing-capable means that the pointer can be used as the destination of a pointer cast and then be used in certain ways without violating the rules of C and C++ on what is commonly called "strict aliasing". Generally, this means that if data has one type, it cannot be used through a pointer as another type (e.g., getting the address of a `float` variable, then casting the `float*` to a `unsigned int*` and accessing it through the `unsigned int*`). Strict aliasing is meant to allow a greater degree of optimizations by being capable of knowing certain data types can always be handled in a specific way / with specific instructions.

{% include anchors.html %}
