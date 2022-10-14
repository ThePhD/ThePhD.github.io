---
layout: post
title: "Cuneicode, and the Future of Text in C"
permalink: /the-cuneicode-and-the-future-of-text-in-c
feature-img: "/assets/img/2022/10/suit-header.png"
thumbnail: "/assets/img/2022/10/suit-header.png"
draft: true
tags: [C, Standard, String, Text, Unicode, API Design]
excerpt_separator: <!--more-->
---

Meow. <!--more-->


## Conversion needs: Genericity

ICU gives us most of the things we need to understand how to do a generic API. Let's take a look at its most general-purpose conversion API, `ucnv_convertEx`:

```cpp
U_CAPI void ucnv_convertEx(
	UConverter *targetCnv, UConverter *sourceCnv, // converters describing the encodings
	char **target, const char *targetLimit, // destination
	const char **source, const char *sourceLimit, // source data
	UChar *pivotStart, UChar **pivotSource, UChar **pivotTarget, const UChar *pivotLimit, // pivot
	UBool reset, UBool flush, UErrorCode *pErrorCode); // error code out-parameter
```

There's a lot going on here, but it's got 4 main components.

- The objects describing conversion. Rather than using a bunch of compile-time "Lucky 7" information, all of that information disappears behind opaque structures and runtime description holders of type `UConverter`.
- The destination data. It's effectively using `char` as a byte-based buffer. It uses a pointer-to-pointer for the "start" pointer because that pointer is going to be advanced up. The second pointer is the end of the byte buffer.
- The source data. It's the same as the destination, but `const`-qualified.
- The pivot. We'll discuss this more down below.
- The error code out-parameter. It could be used as a return values, but C and C++ people are allergic to remembering to check return values for some reason. Out parameter error values are one way to force developers to care, because the function cannot be called without it (at the cost of maybe the tiniest, most insignificant performance impact for the potential single indirect write).

The first 3 bullets describe what we generally need. The buffers have to be type-erased, which means either providing `void*` or using the aliasing-capable `char*` or `unsigned char*`. (Aliasing is when a pointer to one type is used to look at the data of a fundamentally different type; only `char` and `unsigned char` can do that, and `std::byte` if C++ is on the table.) After we type-erase the buffers so that we can work on a "byte" level, we then need to develop what ICU calls `UConverter`s. Converters effectively handle converting between their desired representation (e.g., SHIFT-JIS or EUC-KR) and transport to a given neutral middle-ground encoding (such as UTF-32, UTF-16, or UTF-8). In the case of ICU, they convert to `UChar` objects, which are at-least 16-bit sized objects which can hold UTF-16 code units for UTF-16 encoded data. This becomes the Unicode-based anchor through which all communication happens, and why it is named the "pivot".


### Pivoting: Getting from A to B, through C

ICU is not the first library to come up with this. Featured in libiconv, [libogonek](https://github.com/libogonek/ogonek), my own libraries, encoding_rs, and more, APIs have been using this "pivoting" technique for coming up on a decade and a half now. It is effectively the same platonic ideal of "so long as there is a common, universal encoding that can handle the input data, we will make sure there is an encoding route from A to this ideal intermediate, and then go to B through said intermediate". Let's take a look at `ucnv_convertEx` from ICU again:

```cpp
U_CAPI void ucnv_convertEx (UConverter *targetCnv, UConverter *sourceCnv,
	char **target, const char *targetLimit,
	const char **source, const char *sourceLimit,
	UChar *pivotStart, UChar **pivotSource, UChar **pivotTarget, const UChar *pivotLimit, // ❗❗ Here ❗❗, the pivot
	UBool reset, UBool flush, UErrorCode *pErrorCode);
```

What we see here is that we have the typical destination and source buffers (fashioned as bytes through `char`), but an interesting addendum that drastically improves what we're working with: the addition of an explicit "pivot" buffer. The pivot is typed as various levels of `UChar` pointers, where `UChar` is a stand-in for a type wide enough to hold 16 bits (like `uint_least16_t`). More specifically, the `UChar`-based pivot buffer is meant to be the place where *UTF-16 intermediate data is stored when there is no direct conversion between two encodings*. The iconv library has the same idea, except it does not expose the pivot buffer to you. Emphasis mine:

> It provides support for the follow encodings...
>
> … [huuuuge list] …
>
> It can convert from any of these encodings to any other, **through Unicode conversion.**
>
> — [GNU version of libiconv](https://www.gnu.org/software/libiconv/), September 17th, 2022

In fact, depending on what library you use, you can be dealing with a "pivot", "substrate", or "go-between" encoding that usually ends up being one of UTF-8, UTF-16, or UTF-32. Occasionally, non-Unicode pivots can be used as well but they are exceedingly rare as they often do not accommodate characters from **both** sides of the equation, in a way that Unicode does (or gives room to). Still, just because somebody writes a few decades-old libraries and frameworks around it, doesn't necessarily prove that pivots are the most useful technique. So, are pivots actually useful?

When I wrote my [last article](/any-encoding-ever-ztd-text-unicode-cpp#the-result), we used the concept of a UTF-32-based pivot to convert between UTF-8 and Shift-JIS, without either encoding routine knowing about the other. We did this through the use of a [template function (`ztd::text::transcode`)](https://ztdtext.readthedocs.io/en/latest/api/conversions/transcode.html) and two "Lucky 7" objects, coordinating their inputs, the intermediate "pivot", and the final output. All the type checking and manual `static_assert`s for safety was done at compile-time, so that the code could be as good as it could possibly handle. Assuredly, there had to be a way to connect two encodings in a similar manner for the C programming language, without needing excessive macros or putting templates into C, right?

Luckily, we are not the first people to write a C API for this. But the problem is, a lot of the C and C++ APIs are.... well. All flawed, in interesting ways. Which, of course, means it's time for a cursory review of a lot of the old and new APIs for doing work in C and C++.
