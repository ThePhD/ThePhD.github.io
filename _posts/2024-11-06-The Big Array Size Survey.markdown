---
layout: post
title: "The Big Array Size Survey for C"
permalink: /the-big-array-size-survey-for-c
feature-img: "/assets/img/2024/11/survey-header.jpg"
thumbnail: "/assets/img/2024/11/survey-header.jpg"
tags: [C, C standard, 📜]
excerpt_separator: <!--more-->
---

New in C2y is an operator that does something people have been asking us for, for decades:<!--more--> something that computes the size in elements (NOT bytes) of an array-like thing. This is a [great addition and came from the efforts of Alejandro Colamar in N3369](https://www.open-std.org/JTC1/SC22/WG14/www/docs/n3369.pdf), and was voted into C2y during the recently-finished Minneapolis, MN, USA 2024 standardization meeting. But, there's been some questions about whether we chose the right name or not, and rather than spend an endless amount of Committee time bikeshedding and arguing about this, I wanted to put this question to you, the user, with a survey! (Link to the survey at the bottom of the article.)




# The Operator

Before we get to the survey (link at the bottom), the point of this article is to explain the available choices so you, the user, can make a more informed decision. The core of this survey is to provide a built-in, language name to the behavior of the following macro named `SIZE_KEYWORD`:

```cpp
#define SIZE_KEYWORD(...) (sizeof(__VA_ARGS__) / sizeof(*(__VA_ARGS__)))

int main () {
	int arfarf[] = { 0, 1, 2, 3, 4, 5, 6, 7, 8, 9};
	return SIZE_KEYWORD(arfarf); // same as: `return 10;`
}
```

This is called `nitems()` in BSD-style C, `ARRAY_SIZE()` by others in C with macros, `_countof()` in MSVC-style C, `std::size()` (a library feature) and `std::extent_v<...>` in C++, `len()` in Python, `ztdc_size()` in my personal C library, `extent` in Fortran and other language terminology, and carries many other names both in different languages but also in C itself.

The survey here is not for the naming of a library-based macro (though certain ways of accessing this functionality could be through a macro): there is consensus in the C Standard Committee to make this a normal in-language operator so we can build type safety directly into the language operator rather than come up with increasingly hideous uses of `_Generic` to achieve the same goal. This keeps compile-times low and also has the language accept responsibility for things that it, honestly, should've been responsible for since 1985.

This is the basic level of knowledge you need to access the survey and answer. Further below is an explanation of each important choice in the survey related to the technical features. We encourage you to read this whole blog article before accessing the survey to understand the rationale. The link is at the bottom of this article.




# The Choices

The survey has a few preliminary questions about experience level and current/past usage of C; this does not necessarily change how impactful your choice selection will be! It just might reveal certain trends or ideas amongst certain subsets of individuals. It is also not meant to be extremely specific or even all that deeply accurate. Even if you're not comfortable with C, but you are forced to use it at your Day Job because Nobody Else Will Do This Damn Work, well. You may not like it, but that's still "Professional / Industrial" C development!

The core part of the survey, however, revolve around 2 choices:

- the **usage** pattern required to get to said operator/keyword;
- and, the **spelling** of the operator/keyword itself.

There's several spellings, and three usage patterns. We'll elucidate the usage patterns first, and then discuss the spellings. Given this paper and feature were already accepted to C2y, but that C2y has only JUST started and is still in active development, the goal of this survey is to determine if the community has any sort of preference for the spelling of this operator. Ideally, it would have been nice if people saw the papers in the WG14 document log and made their opinions known ahead-of-time, but this time I am doing my best to reach out to every VIA this article and the survey that is linked at the bottom of the article.




# Usage Pattern

Using `SIZE_KEYWORD` like in the first code sample, this section will explain the three usage patterns and their pros/cons. The program is always meant to return `42`.

```cpp
const double barkbark[] = { 0.0, 0.5, 7.0, 14.7, 23.3, 42.0 };

static_assert(SIZE_KEYWORD(barkbark) == 6, "must have a size of 6");

int main () {
	return (int)barkbark[SIZE_KEYWORD(barkbark) - 1];
}
```



## Underscore and capital letter `_Keyword`; Macro in Header

This technique is a common, age-old way of providing a feature in C. It avoids clobbering the global user namespace with a new keyword that could be affected by user-defined or standards-defined macros (from e.g. POSIX or that already exist in your headers). A keyword still exists, but it's spelled with an underscore and a capital letter to prevent any failures. The user-friendly, lowercase name is only added through a new macro in a new header, so as to prevent breaking old code. Some notable features that USED to be like this:

- `static_assert`/`_Static_assert` with `<assert.h>`
- `_Alignof`/`alignof` with `<stdalignof.h>`
- `_Thread_local`/`thread_local` with `<threads.h>`
- `_Bool`/`bool` with `<stdbool.h>`

As an example, it would look like this:

```cpp
#include <stdkeyword.h>

const double barkbark[] = { 0.0, 0.5, 7.0, 14.7, 23.3, 42.0 };

static_assert(keyword_macro(barkbark) == 6, "must have a size of 6");

int main () {
	return (int)barkbark[_Keyword(barkbark) - 1];
}
```



## Underscore and capital letter `_Keyword`; No Macro in Header

This is a newer way of providing functionality where no effort is made to provide a nice spelling. It's not used very often, except in cases where people expect that the spelling won't be used often or the lowercase name might conflict with an important concept that others deem too important to take for a given spelling. This does not happen often in C, and as such there's really only one prominent example that exists in the standard outside of extensions:

- `_Generic`; no macro ever provided in a header

As an example, it would look like this:

```cpp
// no header
const double barkbark[] = { 0.0, 0.5, 7.0, 14.7, 23.3, 42.0 };

static_assert(_Keyword(barkbark) == 6, "must have a size of 6");

int main () {
	return (int)barkbark[_Keyword(barkbark) - 1];
}
```



## Lowercase `keyword`; No Macro in Header

This is the more bolder way of providing functionality in the C programming language. Oftentimes, this does not happen in C without a sister language like C++ bulldozing code away from using specific lowercase identifiers. It can also happen if a popular extension dominates the industry and makes it attractive to keep a certain spelling. Technically, everyone acknowledges that the lowercase spelling is what we want in most cases, but we settle for the other two solutions because adding keywords of popular words tends to break somebody's code. That leads to a lot of grumbling and pissed off developers who view code being "broken" in this way as an annoying busywork task added onto their workloads. For C23, specifically, a bunch of things were changed from the `_Keyword` + macro approach to using the lowercase name since C++ has already effectively turned them into reserved names:

- `true`, `false`, and `bool`
- `thread_local`
- `static_assert`
- `alignof`
- `typeof` (already an existing extension in many places)

As an example, it would look like this:

```cpp
// no header
const double barkbark[] = { 0.0, 0.5, 7.0, 14.7, 23.3, 42.0 };

static_assert(keyword(barkbark) == 6, "must have a size of 6");

int main () {
	return (int)barkbark[keyword(barkbark) - 1];
}
```




# Keyword Spellings

By far the biggest war over this is not with the usage pattern of the feature, but the actual spelling of the keyword. This prompted a survey from engineer Chris Bazley at ARM, who published his results in [N3350 Feedback for C2y - Survey results for naming of new `nelementsof()` operator](https://www.open-std.org/JTC1/SC22/WG14/www/docs/n3350.pdf). The survey here is not going to query the same set of names, but only the names that seemed to have the most discussion and support in the various e-mails, Committee Meeting discussion, and other drive-by social media / Hallway talking people have done.

Most notably, these options are presented as containing both the lowercase keyword name and the uppercase capital letter `_Keyword` name. Specific combinations of spelling and usage pattern can be given later during an optional question in the survey, along with any remarks you'd like to leave at the end in a text box that can handle a fair bit of text. There are only 6 names, modeled after the most likely spellings similar to the `sizeof` operator. If you have another name you think is REALLY important, please add it at the end of the comments section. Some typical names not included with the reasoning:

- `size`/`SIZE` is too close to `sizeof` and this is not a library function; it would also bulldoze over pretty much every codebase in existence and jeopardize other languages built on top of / around C.
- `nitems`/`NITEMS` is a BSD-style way of spelling this and we do not want to clobber that existing definition.
- `ARRAY_SIZE`/`stdc_size` and similar renditions are not provided because this is an operator exposed through a keyword and not a macro, but even then `array_size`/`_Array_size` were deemed too awkward to spell.
- `dimsof`/`dimensionsof` was, similarly, not all that popular and `dimensions` as a word did not convey the meaning very appropriately to begin with.
- Other brave but unfortunately unmentioned spellings that did not make the cut.

The options in the survey are as below:



## `lenof` / `_Lenof`

A very short spelling that utilizes the word "length", but shortened in the typical C fashion. Very short and easy to type, and it also fits in with most individual's idea of how this works. It is generally favored amongst C practitioners, and is immediately familiar to Pythonistas. A small point of contention: doing `_Lenof(L"barkbark")` produces the answer "5", not "4" (the null terminator is counted, just as in `sizeof("barkbark")`). This has led some to believe this would result in "confusion" when doing string processing. It's unclear whether this worry is well-founded in any data and not just a nomenclature issue.

As "len" and `lenof` are popular in C code, this one would likely need a underscore-capital letter keyword and a macro to manage its introduction, but it is short.

```cpp
const double barkbark[] = { 0.0, 0.5, 7.0, 14.7, 23.3, 42.0 };

static_assert(_Lenof(barkbark) == 6, "must have an length of 6");

int main () {
	return (int)barkbark[lenof(barkbark) - 1];
}
```



## `lengthof` / `_Lengthof`

This spelling won in Chris Bazley's ARM survey of the 40 highly-qualified C/C++ engineers and is popular in many places. Being spelled out fully seems to be of benefit and heartens many users who are sort of sick of a wide variety of C's crunchy, forcefully shortened spellings like `creat` (or `len`, for that matter, though `len` is much more understood and accepted). It is the form that was voted into C2y as `_Lengthof`, though it's noted that the author of the paper that put `_Lengthof` into C is strongly against its existence and thinks this choice will encourage off-by-one errors (similarly to `lenof` discussed above). Still, it seems like both the least hated and most popular among the C Committee and the adherents who had responded to Alejandro Colomar's GCC patch for this operator. Whether it will continue to be popular with the wider community has yet to be seen.

As "length" and `lengthof` are popular in C code, this one would likely need a underscore-capital letter keyword and a macro to introduce it carefully into existing C code.

```cpp
const double barkbark[] = { 0.0, 0.5, 7.0, 14.7, 23.3, 42.0 };

static_assert(_Lengthof(barkbark) == 6, "must have an length of 6");

int main () {
	return (int)barkbark[lengthof(barkbark) - 1];
}
```



## `countof` / `_Countof`

This spelling is a favorite of many people who want a word shorter than `length` but still fully spelled out that matches its counterpart `size`/`sizeof`. It has strong existing usage in codebases around the world, including a definition of this macro in Microsoft's C library. It's favored by a few on the C Committee, and I also received an e-mail about `COUNT` being provided by the C library as a macro. It was, unfortunately, not polled in the ARM survey. It also conflicts with C++'s idea of `count` as an algorithm rather than an operation (C++ just uses `size` for counting the number of elements). It is dictionary-definition accurate to what this feature is attempting to do, and does not come with off-by-one concerns associated with strings and "length", typically.

As "count" and `countof` are popular in C code, this too would need some management in its usage pattern to make it available everywhere without getting breakage in some existing code.

```cpp
const double barkbark[] = { 0.0, 0.5, 7.0, 14.7, 23.3, 42.0 };

static_assert(_Countof(barkbark) == 6, "must have an length of 6");

int main () {
	return (int)barkbark[countof(barkbark) - 1];
}
```



## `nelemsof` / `_Nelemsof`

This spelling is an alternative spelling to `nitems()` from BSD (to avoid taking `nitems` from BSD). `nelems` is also seem as the short, cromulent spelling of another suggestion in this list, `nelementsof`. It is a short spelling but lacks spaces between `n` and `elems`, but emphasizes this is the number of elements being counted and not anything else. The `n` is seen as a universal letter for the count of things, and most people who encounter it understand it readily enough. It lacks problems about off-by-one counts by not being associated with strings in any manner, though `n` being a common substitution for "length" might bring this up in a few people's minds.

As "nelems" and `nelems` are popular in C code, this too would need some management in its usage pattern to make it available everywhere without getting breakage in some existing code.

```cpp
const double barkbark[] = { 0.0, 0.5, 7.0, 14.7, 23.3, 42.0 };

static_assert(_Nelems(barkbark) == 6, "must have an length of 6");

int main () {
	return (int)barkbark[nelems(barkbark) - 1];
}
```



## `nelementsof` / `_Nelementsof`

This is the long spelling of the `nelems` option just prior. It is the preferred name of the author of N3369, Alejandro Colomar, before WG14 worked to get consensus to change the name to `_Lengthof` for C2y. It's a longer name that very clearly states what it is doing, and all of the rationale for `nelems` applies.

This is one of the only options that has a name so long and unusual that it shows up absolutely nowhere that matters. It can be standardized without fear as `nelements` with no macro version whatsoever, straight up becoming a keyword in the Core C language without any macro/header song-and-dance.

```cpp
const double barkbark[] = { 0.0, 0.5, 7.0, 14.7, 23.3, 42.0 };

static_assert(nelementsof(barkbark) == 6, "must have an length of 6");

int main () {
	return (int)barkbark[nelementsof(barkbark) - 1];
}
```



## `extentof` / `_Extentof`

During the discussion of the paper in the Minneapolis 2024 meeting, there was a surprising amount of in-person vouching for the name `extentof`. They also envisioned it coming with a form that allowed to pass in which dimension of a multidimensional array you wanted to get the extent of, similar to C++'s `std::extent_v` and `std::rank_v`, as seen [here](https://en.cppreference.com/w/cpp/types/extent) and [here](https://en.cppreference.com/w/cpp/types/rank). Choosing this name comes with the implicit understanding that additional work would be done to furnish a `rankof`/`_Rankof` (or similar spelling) operator for C as well in some fashion to allow for better programmability over multidimensional arrays. This option tends to appeal to Fortran and Mathematically-minded individuals in general conversation, and has a certain appeal among older folks for some reason I have not been able to appropriately pin down in my observations and discussions; whether or not this will hold broadly in the C community is anyone's guess.

As extent is a popular word and `extentof` similarly, this one would likely need a macro version with an underscore capital-letter keyword, but the usage pattern can be introduced gradually and gracefully.

```cpp
const double barkbark[] = { 0.0, 0.5, 7.0, 14.7, 23.3, 42.0 };

static_assert(_Extentof(barkbark) == 6, "must have an extent of 6");

int main () {
	return (int)barkbark[extentof(barkbark) - 1];
}
```



# The Survey

**[Here's the survey: https://www.allcounted.com/s?did=qld5u66hixbtj&lang=en_US](https://www.allcounted.com/s?did=qld5u66hixbtj&lang=en_US).**

There is an optional question at the end of the survey, before the open-ended comments, that allows for you to also rank and choose very specific combinations of spelling and feature usage mechanism. This allows for greater precision beyond just answering the two core questions, if you want to explain it.


### Employ your democratic right to have a voice and inform the future of C, today!

Good Luck! 💚



- Banner and Title Photo by [Luka, from Pexels](https://www.pexels.com/photo/close-up-photo-of-survey-spreadsheet-590022/)

{% include anchors.html %}
