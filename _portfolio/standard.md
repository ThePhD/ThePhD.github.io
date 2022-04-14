---
layout: post
title: Standard C and C++ Proposals
feature-img: "assets/img/portfolio/standard.png"
img: "assets/img/portfolio/standard.png"
date: May 21st, 2021
tags: [C, C++, proposals, standard, Portfolio, 🚌, ⌨️]
---

I have quite a few C++ proposals I am working on and percolating through the C++ Standards Committee. You will find the submitted and unsubmitted ones below! Be sure to also check out the `proposals` and `standard` tags for any writeups related to C++ proposals!


### Actively Pursued Evolution Proposals:

C++ | p1040: std::embed | [Published](https://wg21.link/p1040) | [Latest Draft](/_vendor/future_cxx/papers/d1040.html)
C++ | p1629: Standard Text Encoding | [Published](https://wg21.link/p1629) | [Latest Draft](/_vendor/future_cxx/papers/d1629.html)
C++ | p1664: Reconstructible Ranges | [Published](https://wg21.link/p1664) | [Latest Draft](/_vendor/future_cxx/papers/d1664.html)
C++ | p1967: Preprocessor embed - Binary Resource Inclusion | [Published](https://wg21.link/p1967) | [Latest Draft](/_vendor/future_cxx/papers/d1967.html)
C++ | p2513: `char8_t` Compatibility and Portability Fixes | [Published](https://wg21.link/p2513) | [Latest Draft](/_vendor/future_cxx/papers/d2513.html)
C   | n2961: Literal Suffixes for `size_t` | [Published](http://www.open-std.org/jtc1/sc22/wg14/www/docs/n2961.htm) | [Latest Draft](/_vendor/future_cxx/papers/C%20-%20Literal%20Suffixes%20for%20size_t.html)
C   | n2962: `__supports_literal` | [Published](http://www.open-std.org/jtc1/sc22/wg14/www/docs/n2962.htm) | [Latest Draft](/_vendor/future_cxx/papers/C%20-%20__supports_literal.html)
C   | n2963: Enhanced Enumerations | [Published](http://www.open-std.org/jtc1/sc22/wg14/www/docs/n2963.htm) | [Latest Draft](/_vendor/future_cxx/papers/C%20-%20Enhanced%20Enumerations.html)
C   | n2964: Improved Normal Enumerations | [Published](http://www.open-std.org/jtc1/sc22/wg14/www/docs/n2964.htm) | [Latest Draft](/_vendor/future_cxx/papers/C%20-%20Improved%20Normal%20Enumerations.html)
C   | n2965: Modern Bit Utilities | [Published](http://www.open-std.org/jtc1/sc22/wg14/www/docs/n2965.htm) | [Latest Draft](/_vendor/future_cxx/papers/C%20-%20Modern%20Bit%20Utilities.html)
C   | n2966: Restartable and Non-Restartable Functions for Efficient Character Conversions | [Published](http://www.open-std.org/jtc1/sc22/wg14/www/docs/n2966.htm) | [Latest Draft](/_vendor/future_cxx/papers/C%20-%20Efficient%20Character%20Conversions.html)
C   | n2967: Preprocessor embed - Binary Resource Inclusion | [Published](http://www.open-std.org/jtc1/sc22/wg14/www/docs/n2967.htm) | [Latest Draft](/_vendor/future_cxx/papers/C%20-%20embed.html)
C   | n2968: Prefixes for the Standard Library | [Published](http://www.open-std.org/jtc1/sc22/wg14/www/docs/n2968.htm) | [Latest Draft](/_vendor/future_cxx/papers/C%20-%20Prefixes%20for%20the%20Standard%20Library.html)
C   | n29XX: Transparent Function Aliases | [Published (Old)](http://www.open-std.org/jtc1/sc22/wg14/www/docs/n2901.htm) | [Latest Draft](/_vendor/future_cxx/papers/C%20-%20Transparent%20Function%20Aliases.html)


### Approved Proposals:

Most have been completed / merged at this point! The rest of the work is up above!


### Completed / Merged Proposals:

C++ | p1025: Update References to the Unicode Standard | [Published](https://wg21.link/p1025) | [Accepted, C++20!](https://wg21.link/p1025)
C++ | p1682: `std::to_underlying(T enum_value)` | [Published](https://wg21.link/p1682) | [Accepted, C++23!](/_vendor/future_cxx/papers/d1682.html)
C++ | p1132: out_ptr - a scalable output pointer abstraction | [Published](https://wg21.link/p1132) | [Accepted, C++23!](/_vendor/future_cxx/papers/d1132.html)
C++ | p0330: Literal Suffixes for `ptrdiff_t` and `size_t` | [Published](https://wg21.link/p0330) | [Accepted, C++23!](/_vendor/future_cxx/papers/d0330.html)
C++ | p1301: [[nodiscard("should have a reason")]] | [Published](https://wg21.link/p1301) | [Accepted, C++20!](/_vendor/future_cxx/papers/d1301.html)
C   | n2448: [[nodiscard("should have a reason")]] | [Published](http://www.open-std.org/jtc1/sc22/wg14/www/docs/n2448.pdf) | [Accepted, C23!](/_vendor/future_cxx/papers/C%20-%20nodiscard.html)
C   | n2594: Mixed Wide String Literal Concatenation | [Published](http://www.open-std.org/jtc1/sc22/wg14/www/docs/n2594.htm) | [Accepted, C23!](/_vendor/future_cxx/papers/C%20-%20Mixed%20Wide%20String%20Literal%20Concatenation.html)
C   | n2726: `_Imaginary_I` and `_Complex_I` Qualifiers | [Published](http://www.open-std.org/jtc1/sc22/wg14/www/docs/n2726.htm) | [Accepted, C23!](/_vendor/future_cxx/papers/C%20-%20_Imaginary_I%20and%20_Complex_I%20Qualifiers.html)
C   | n2728: `char16_t` & `char32_t` string literals shall be UTF-16 and UTF-32 | [Published](http://www.open-std.org/jtc1/sc22/wg14/www/docs/n2728.htm) | [Accepted, C23!](/_vendor/future_cxx/papers/C%20-%20char16_t%20&%20char32_t%20string%20literals%20shall%20be%20UTF-16%20&%20UTF-32.html)
C   | n2927: Not-So-Magic: `typeof()` for C | [Published](http://www.open-std.org/jtc1/sc22/wg14/www/docs/n2927.htm) | [Accepted, C23!](/_vendor/future_cxx/papers/C%20-%20typeof.html)
C   | n2828: Unicode Sequences More Than 21 Bits Are A Constraint Violation | [Published](http://www.open-std.org/jtc1/sc22/wg14/www/docs/n2828.htm) | [Accepted, C23!](/_vendor/future_cxx/papers/C%20-%20Unicode%20Sequences%20More%20Than%2021%20Bits%20are%20a%20Constraint%20Violation.html)
C   | n2900: Consistent, Warningless, and Intuitive Initialization with `{}` | [Published](http://www.open-std.org/jtc1/sc22/wg14/www/docs/n2900.htm) | [Accepted, C23!](/_vendor/future_cxx/papers/C%20-%20Consistent,%20Warningless,%20and%20Intuitive%20Initialization%20with%20%7B%7D.html)


### Being Reworked from Feedback:

C++ | p1214: Callables Cleanup! - Overloading and Pointer-to-Members | [Published](https://wg21.link/p1214) | [Latest Draft](/_vendor/future_cxx/papers/d1214.html)
C++ | p1039: I got you, FAM - Flexible Array Members for C++| [Published](https://wg21.link/p1039) | [Latest Draft](/_vendor/future_cxx/papers/d1039.html)


### Educational

These proposals are educational in nature only and, while making recommendations, will not see additional pursuit or wording.

C++ | p1683: A Survey of References for Standard Library Types - A Survey of Optionals | [Published](https://wg21.link/p1683) | [Latest Draft](/_vendor/future_cxx/papers/d1683.html)


### Expired:

These proposals are dead and will not be picked up again.


C++ | p1175: A simple and practical optional reference for C++ | [Published](https://wg21.link/p1175) | [Latest Draft](/_vendor/future_cxx/papers/d1175.html)
C++ | p1130: Module Resource Requirement Propagation | [Published](https://wg21.link/p1130) | [Latest Draft](https://thephd.dev/_vendor/future_cxx/papers/d1130.html)


### Cannot Pursue

These proposals are challenging and may require significant rework or language enhancement. Some have also been superseded by other proposals.

C++ | p0051: `std::overload` | Not Published | [Latest Draft](/_vendor/future_cxx/papers/d0051.html)
C++ | p1803: packexpr(args, I) -- compile-time friendly pack inspection | [Published](https://wg21.link/p1803) | [Latest Draft](/_vendor/future_cxx/papers/d1803.html)
C++ | p1193: Explicitly Specified Returns for (Implicit) Conversions | [Published](https://wg21.link/p1193) | [Latest Draft](/_vendor/future_cxx/papers/d1193.html)
C++ | p1378: `consteval is_string_literal(T*)` | Not Published | [Latest Draft](/_vendor/future_cxx/papers/d1378.html)
