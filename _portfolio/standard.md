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

C++29   | p1629: Standard Text Encoding | [Published](https://wg21.link/p1629) | [Latest Draft](/_vendor/future_cxx/papers/d1629.html)
C++29   | p1664: Reconstructible Ranges | [Published](https://wg21.link/p1664) | [Latest Draft](/_vendor/future_cxx/papers/d1664.html)
C++26   | p1967: Preprocessor `embed` - Binary Resource Inclusion | [Published](https://wg21.link/p1967) | [Latest Draft](/_vendor/future_cxx/papers/d1967.html)
C2y/C3a | n2998: Literal Suffixes for `size_t` | [Published](https://www.open-std.org/jtc1/sc22/wg14/www/docs/n2998.htm) | [Latest Draft](/_vendor/future_cxx/papers/C%20-%20Literal%20Suffixes%20for%20size_t.html)
C2y/C3a | n3199: Improved `__attribute__((cleanup))` Through `defer` | [Published](https://www.open-std.org/jtc1/sc22/wg14/www/docs/n3199.htm) | [Latest Draft](/_vendor/future_cxx/papers/C%20-%20Improved%20__attribute__((cleanup))%20Through%20defer.html)
C2y/C3a | n3329: Transparent Aliases | [Published](https://www.open-std.org/jtc1/sc22/wg14/www/docs/n3329.htm) | [Latest Draft](/_vendor/future_cxx/papers/C%20-%20Transparent%20Aliases.html)
C2y/C3a | n3331: `_Generic` Realignment and Improvement | [Published](https://www.open-std.org/jtc1/sc22/wg14/www/docs/n3331.htm) | [Latest Draft](/_vendor/future_cxx/papers/C%20-%20_Generic%20Realignment%20and%20Improvement.html)
C2y/C3a | n3332: `_Record` types | [Published](https://www.open-std.org/jtc1/sc22/wg14/www/docs/n3332.htm) | [Latest Draft](/_vendor/future_cxx/papers/C%20-%20_Record%20types.html)
C2y/C3a | n3333: Integer Constant Expression Initialized `const` Integer Declarations are Implicitly `constexpr` | [Published](https://www.open-std.org/jtc1/sc22/wg14/www/docs/n3333.htm) | [Latest Draft](/_vendor/future_cxx/papers/C%20-%20Initialized%20const%20Integer%20Declarations.html)

C2y/C3a | cXXX3: Comprehensive Statement Expressions | Soon™ | [Latest Draft](/_vendor/future_cxx/papers/C%20-%20Comprehensive%20Statement%20Expressions.html)
C2y/C3a | cXXX4: Functional Functions - A Comprehensive Proposal Overviewing Blocks, Nested Functions, and Lambdas for C | Soon™ | [Latest Draft](/_vendor/future_cxx/papers/C%20-%20Functional%20Functions.html)
C2y/C3a | cXXX5: `-fwrapv` for Everyone - A Measured Approach to `_Operator`s | Soon™ | [Latest Draft](/_vendor/future_cxx/papers/C%20-%20A%20Measured%20Approach%20to%20_Operators.html)
C2y/C3a | cXXX7: `T[..; …]` — A Non-owning Sized Arrays for Spans and Beyond | Soon™ | [Latest Draft](/_vendor/future_cxx/papers/C%20-%20Non-owning%20Sized%20Arrays.html)
C2y/C3a | cXXX9: Variable-Length Array (VLA) Types for Any Size | Soon™ | [Latest Draft](/_vendor/future_cxx/papers/C%20-%20Variable-Length%20Array%20(VLA)%20Types%20for%20Any%20Size.html)
C2y/C3a | cXX10: Variable-Length Array (VLA) Allocator Control | Soon™ | [Latest Draft](/_vendor/future_cxx/papers/C%20-%20Variable-Length%20Array%20(VLA)%20Allocation%20Control.html)


### Approved Proposals:

Most have been completed / merged at this point! The rest of the work is up above!


### Completed / Merged Proposals:

C++20 | p1025: Update References to the Unicode Standard | [Published](https://wg21.link/p1025) | [Accepted, C++20!](https://wg21.link/p1025)
C++20 | p1301: `[[nodiscard("should have a reason")]]` | [Published](https://wg21.link/p1301) | [Accepted, C++20!](/_vendor/future_cxx/papers/d1301.html)
C++23 | p1682: `std::to_underlying(T enum_value)` | [Published](https://wg21.link/p1682) | [Accepted, C++23!](/_vendor/future_cxx/papers/d1682.html)
C++23 | p1132: `out_ptr` - a scalable output pointer abstraction | [Published](https://wg21.link/p1132) | [Accepted, C++23!](/_vendor/future_cxx/papers/d1132.html)
C++23 | p0330: Literal Suffixes for `ptrdiff_t` and `size_t` | [Published](https://wg21.link/p0330) | [Accepted, C++23!](/_vendor/future_cxx/papers/d0330.html)
C++23 | p2513: `char8_t` Compatibility and Portability Fixes | [Published](https://wg21.link/p2513) | [Accepted, C++23!](/_vendor/future_cxx/papers/d2513.html)
C++26 | p2537: Relax `va_start` Requirements to Match C | [Published](https://wg21.link/p2537) | [Latest Draft](/_vendor/future_cxx/papers/d2537.html)
C23   | n2448: `[[nodiscard("should have a reason")]]` | [Published](https://www.open-std.org/jtc1/sc22/wg14/www/docs/n2448.pdf) | [Accepted, C23!](/_vendor/future_cxx/papers/C%20-%20nodiscard.html)
C23   | n2594: Mixed Wide String Literal Concatenation | [Published](https://www.open-std.org/jtc1/sc22/wg14/www/docs/n2594.htm) | [Accepted, C23!](/_vendor/future_cxx/papers/C%20-%20Mixed%20Wide%20String%20Literal%20Concatenation.html)
C23   | n2968: Prefixes for the Standard Library | [Published](https://www.open-std.org/jtc1/sc22/wg14/www/docs/n2968.htm) | [Accepted, C23!](/_vendor/future_cxx/papers/C%20-%20Prefixes%20for%20the%20Standard%20Library.html)
C23   | n2726: `_Imaginary_I` and `_Complex_I` Qualifiers | [Published](https://www.open-std.org/jtc1/sc22/wg14/www/docs/n2726.htm) | [Accepted, C23!](/_vendor/future_cxx/papers/C%20-%20_Imaginary_I%20and%20_Complex_I%20Qualifiers.html)
C23   | n2728: `char16_t` & `char32_t` string literals shall be UTF-16 and UTF-32 | [Published](https://www.open-std.org/jtc1/sc22/wg14/www/docs/n2728.htm) | [Accepted, C23!](/_vendor/future_cxx/papers/C%20-%20char16_t%20&%20char32_t%20string%20literals%20shall%20be%20UTF-16%20&%20UTF-32.html)
C23   | n2927: Not-So-Magic: `typeof()` for C | [Published](https://www.open-std.org/jtc1/sc22/wg14/www/docs/n2927.htm) | [Accepted, C23!](/_vendor/future_cxx/papers/C%20-%20typeof.html)
C23   | n2828: Unicode Sequences More Than 21 Bits Are A Constraint Violation | [Published](https://www.open-std.org/jtc1/sc22/wg14/www/docs/n2828.htm) | [Accepted, C23!](/_vendor/future_cxx/papers/C%20-%20Unicode%20Sequences%20More%20Than%2021%20Bits%20are%20a%20Constraint%20Violation.html)
C23   | n2900: Consistent, Warningless, and Intuitive Initialization with `{}` | [Published](https://www.open-std.org/jtc1/sc22/wg14/www/docs/n2900.htm) | [Accepted, C23!](/_vendor/future_cxx/papers/C%20-%20Consistent,%20Warningless,%20and%20Intuitive%20Initialization%20with%20%7B%7D.html)
C23   | n3011: Oops, Compound Literals and Empty Initializers | [Published](https://www.open-std.org/jtc1/sc22/wg14/www/docs/n3011.htm) | [Accepted, C23!](/_vendor/future_cxx/papers/C%20-%20Oops,%20Compound%20Literals%20with%20Empty%20Initializers.html)
C23   | n3017: Preprocessor `embed` - Binary Resource Inclusion | [Published](https://www.open-std.org/jtc1/sc22/wg14/www/docs/n3017.htm) | [Accepted, C23!](/_vendor/future_cxx/papers/C%20-%20embed.html)
C23   | n3022: Modern Bit Utilities (Most Components) | [Published](https://www.open-std.org/jtc1/sc22/wg14/www/docs/n3022.htm) | [Accepted, C23!](/_vendor/future_cxx/papers/C%20-%20Modern%20Bit%20Utilities.html)
C23   | n3029: Improved Normal Enumerations | [Published](https://www.open-std.org/jtc1/sc22/wg14/www/docs/n3029.htm) | [Accepted, C23!](/_vendor/future_cxx/papers/C%20-%20Improved%20Normal%20Enumerations.html)
C23   | n3030: Enhanced Enumerations | [Published](https://www.open-std.org/jtc1/sc22/wg14/www/docs/n3030.htm) | [Accepted, C23!](/_vendor/future_cxx/papers/C%20-%20Enhanced%20Enumerations.html)
C2y   | n3330: More Modern Bit Utilities | [Published](https://www.open-std.org/jtc1/sc22/wg14/www/docs/n3330.htm) | [Accepted, C2y!](/_vendor/future_cxx/papers/C%20-%20More%20Modern%20Bit%20Utilities.html)
C2y   | n3265: Restartable Functions for Efficient Character Conversions | [Published](https://www.open-std.org/jtc1/sc22/wg14/www/docs/n3265.htm) | [Accepted, C2y!](/_vendor/future_cxx/papers/C%20-%20Efficient%20Character%20Conversions.html)


### Being Reworked from Feedback:

C++?? | p1214: Callables Cleanup! - Overloading and Pointer-to-Members | [Published](https://wg21.link/p1214) | [Latest Draft](/_vendor/future_cxx/papers/d1214.html)
C++?? | p1039: I got you, FAM - Flexible Array Members for C++| [Published](https://wg21.link/p1039) | [Latest Draft](/_vendor/future_cxx/papers/d1039.html)
C++?? | p1040: `std::embed` | [Published](https://wg21.link/p1040) | [Latest Draft](/_vendor/future_cxx/papers/d1040.html)


### Educational

These proposals are educational in nature only and, while making recommendations, will not see additional pursuit or wording.

C++ | p1683: A Survey of References for Standard Library Types - A Survey of Optionals | [Published](https://wg21.link/p1683) | [Latest Draft](/_vendor/future_cxx/papers/d1683.html)


### Expired:

These proposals are dead and will not be picked up again.


C++ | p1175: A simple and practical optional reference for C++ | [Published](https://wg21.link/p1175) | [Latest Draft](/_vendor/future_cxx/papers/d1175.html)
C++ | p1130: Module Resource Requirement Propagation | [Published](https://wg21.link/p1130) | [Latest Draft](https://thephd.dev/_vendor/future_cxx/papers/d1130.html)
C   | n3198: Unwinding | [Published](https://www.open-std.org/jtc1/sc22/wg14/www/docs/n3198.htm) | [Latest Draft](/_vendor/future_cxx/papers/C%20-%20Unwinding.html)
C   | n2995: `__supports_literal` | [Published](https://www.open-std.org/jtc1/sc22/wg14/www/docs/n2995.htm) | [Latest Draft](/_vendor/future_cxx/papers/C%20-%20__supports_literal.html)


### Cannot Pursue

These proposals are challenging and may require significant rework or language enhancement. Some have also been superseded by other proposals.

C++ | p0051: `std::overload` | Not Published | [Latest Draft](/_vendor/future_cxx/papers/d0051.html)
C++ | p1803: packexpr(args, I) -- compile-time friendly pack inspection | [Published](https://wg21.link/p1803) | [Latest Draft](/_vendor/future_cxx/papers/d1803.html)
C++ | p1193: Explicitly Specified Returns for (Implicit) Conversions | [Published](https://wg21.link/p1193) | [Latest Draft](/_vendor/future_cxx/papers/d1193.html)
C++ | p1378: `consteval is_string_literal(T*)` | Not Published | [Latest Draft](/_vendor/future_cxx/papers/d1378.html)
C++ | p2958: `typeof` and `typeof_unqual` for C++ | [Published](https://wg21.link/p2958) | [Latest Draft](/_vendor/future_cxx/papers/d2958.html)

{% include anchors.html %}
