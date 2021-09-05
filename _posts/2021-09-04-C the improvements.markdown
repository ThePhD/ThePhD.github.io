---
layout: post
title: "C-ing the Improvement: Progress on C23"
permalink: /c-the-improvements-june-september-virtual-c-meeting
feature-img: "/assets/img/2021/09/cat-eyes.jpg"
thumbnail: "/assets/img/2021/09/cat-eyes.jpg"
tags: [C, Standard, ISO, 📜]
excerpt_separator: <!--more-->
---

Get it? Because see, like in visually perceive the changes we're making? Eh? Ehhhh?!<!--more-->

... Okay fine don't laugh but I thought it was funny. Hmph!

A [long time ago I promised I'd write](/no-us-without-you-elifdef-elifndef-c-n2645) more about what's been going on in the world of C. I finally am not absolutely drowning in work this Labor Day Weekend so let's talk about all of the stuff Committee Members have been up to for making C a better place for everybody! We'll include everything that was accepted and is a "big ticket" item (big as in C big, not C++ big; C++ users will see a lot of this and be like "lol nice late features NERDS" to which I say WE'LL BE AS SLOW AND AS CAREFUL AS WE GODDAMN WANT YOU NOISY WHIPPERSNAPPERS! 👴 ✊ ❕).



## N2645 - `#elifdef` and `#elifndef`

I wrote about [this change before and the power of users](/no-us-without-you-elifdef-elifndef-c-n2645) actually e-mailing people on the Committee and very intently telling us that they value something (we were probably set to go break-even on votes, which was not enough for positive consensus). This one made it into C23 and you should start heckling your implementers to add it (including to older standard flags too, because it's just a Good and Cool feature!). Mostly, if you've ever written:

```cpp
#ifdef FOO
	/* stuff if there is a defined FOO */
#elifdef BAR
	/* stuff if there is a defined BAR */
#elifndef BAZ
	/* stuff if there is NO defined BAZ */
#else
	/* stuff as the last resort */
#endif
```

And then got cryptic errors, this proposal is for you. It makes it so `#ifdef`/`#ifndef` has normal, regular forms in the `#elif` variety as well. Which you think would have just been there since the beginning, but weren't. (Probably because they added the `defined(...)` operator, and just said "well, that's the most generic form we could want, pack it in!".)

Remember, keeping an open communication channel (Twitter, Slack, E-Mail, Literally Normal Mail, Carrier Pigeon) is a good thing. Don't let us get too high on our own Committee Supply.



## N2626 - Digit Separators

Have you ever written 4958409583ull as a numeric constant and felt your eyes cross when trying to read it over and over again? Have you ever wanted things to be a little easier to read when you make complex chunks of bits (separate it into bytes, separate it [into nybbles](https://en.wikipedia.org/wiki/Nibble))? Now you can! Introducing: digit separators! Because we didn't want any cultures getting the idea we loved them anymore than any other culture, we chose a digit separator everyone can ~~hate~~love equally, so it looks like this:

```cpp
const unsigned magical_number = 1'633'902'946;
```

I should also note that C23 will also have [Binary Integer Literals](http://www.open-std.org/jtc1/sc22/wg14/www/docs/n2549.pdf), so the same number can be written out in a more precise grouping of binary as well:

```cpp
const unsigned magical_number = 0b0110'0001'0110'0011'0110'0001'0110'0010;
```

It looks pretty great now, nice! It also matches C++'s syntax, so we thankfully don't have to have do any shenanigans for those of us who need headers and code that compiles between both languages. And since we're talking about binary literals and the like...



## N2630 - Formatted input/output of binary integer numbers

This paper allows us to print binary numbers using the `"%b"` specifier. This is a thing people have also needed from time to time and it made sense after we received binary literals. It applies to `scanf` family of functions, `strto*` family of functions, and the `printf` family of functions. Small caveat: since some implementations already stole `"%B"` for themselves (implementations were allowed to do this, yes!), we do not have a print specification for `"%B"` which would print out the prefix. You're going to need to make do with `"0b%b"` for now. Sorry: blame the existing implementations ¯\\\_(ツ)\_/¯.

But, even if that has you feeling a bit down, we've also got another formatting feature which is actually pretty great!



## N2680 - Specific Width Length Modifiers

Specific Width Length Modifiers was a reaction to what appeared to be a very simple question that unfortunately did not have a straightforward answer. "How do I, securely, print an integer of any size?" Originally, the answer was `printf("%j", (intmax_t)any_integer_ever);`. Unfortunately, ABI has made it so that while implementations have efficient instructions for processing `(u)int128_t` and `(u)int256_t` integer types that could be elevated to the common arithmetic type syntax, we cannot and could not change the type definition of `intmax_t` on existing platforms. (It is, effectively, `(unsigned) long long` until the next huge architecture break between e.g. x86_64 and I dunno, "i128tanium" or something?? (i128tanium is not a real architecture, we are waxing poetical here.)) This presented both a security and a portability issue: how is one supposed to print the integer type they like for their type?

The solution Seacord brought to the table was `printf("%w128d", my_128bit_integer);`. You can now pass exactly the number of bits of your integer type to the formatter. Note that this also means that you can use it to print the "low" bits of an `int` when trying to just read the low-end bits as well, such as by using `printf("%w8d", 0xFEE);`. This gives us a cross-platform way to handle the bits with a given implementation, and correctly print it out! It's also guaranteed to be safe: if the integer supports formatting such an integer of such a specific size, it will do it properly (and otherwise, it will error, as per usual with an unknown formatting string). The standard only requires that it handle sizes equivalent to what it wants to support, asides from mandated support for the `sizeof(built-in-integer) * CHAR_BIT` sizes.



## N2683 - Towards Integer Safety

For a long time, people have been complaining that various bits of integer work are, effectively, undefined behavior. C had no way of preventing it except for users rolling their own very cumbersome and painful libraries and learning all of the quirks on their own (often to the detriment of shipping subtly broken code and crying inside) or relying on compiler flags to solve some problems for them like `-fwrapv`. Now, we know that somebody, somewhere, will complain if we actually change the default behavior of integers to be legitimately safer everywhere ("oh my god what have you done to my PERFORMANCE listen here you standards people my code is safe I'm a GOD TIER programmer take your safety checks and [REDACTED] [REDACTED] your [REDACTED]"). So, we did the usual bit: put it in some library functions, and called it a day. The new `<stdckdint.h>` header is going to be added, with some (macro) functions:

```cpp
#include <stdckdint.h>

bool ckd_add(type1 *result, type2 a, type3 b);
bool ckd_sub(type1 *result, type2 a, type3 b);
bool ckd_mul(type1 *result, type2 a, type3 b);
```

This is modeled after the GCC functions and quite a few existing C and C++ libraries. Notice the use of `type1`, `type2`, and `type3`, rather than any specific integral arguments: that's because these functions are type-generic. They are implemented with macros (and therefore explicitly cannot be redeclared or redefined by the end-user, nor can they be macro-suppressed as stated in §7.1.4 in the C Standard). They should work for all of the built-in integer types.

You may also notice that division isn't on the table: that's because most libraries just quietly left division out of them, including the GCC intrinsics. Why? I'm gonna be straight with you: I'm not exactly sure. What I can tell you is that C is addicted to existing practice, though! And, because none of Java, C, or any others actually included functions to handle division, left shift, or right shift, the proposal didn't include it. ("Can't you approximate those last three with the first three shown here?", I hear you ask. And the answer is: sure, but is that what you'd really like to spend your time on?)

Whether or not you consider the functionality complete, however, doesn't change that the most frequent cases of overflow, underflow, and other shenanigans are from addition, multiplication, and subtraction in the wild. So, these functions are more than enough to cover a grand multitude of CVEs that have been - and currently are - being exploited out in the wild today, which is a great step forward for the baseline of production for C23. Just put the two operands in the last 2 spots, and if the function returns `true`, then `*result` one will be populated with the actual result of the operation since it was safe and no overflow/underflow/etc. happened. If it doesn't return `true`, please don't touch whatever is in `*result` I promise you it will not go well.



## N2709 - Adding a Fundamental Type for N-bit Integers

`_BitInt(N)`, where `N` is the number of bits you want in an integer, is a really, really big deal for C in my opinion. There have been entire extensions built around C for analyzing the compile-time information available to integers in C programs. Most often, that information is used to determine the effective range of integer operations, and strip off those bits for savings in e.g. FPGAs or other custom hardware. Entire optimizations go into "how many bits is this really using and can I get some speed if I rearrange these 4 integers to instead be a smaller set of bytes I can perform parallel instructions with?". Unfortunately, all of that has been stuck fairly fundamentally in the land of "special magic C users cannot hope to touch".

Until now.

`_BitInt(N)` makes C suitable for hardware and embedded work, as it provides something that BitVec Author and Embedded Badass myrrlyn explains succinctly:

> low level (close to hardware) programming requires that numbers very firmly do not spuriously change width or signage, which is behavior that c actively chooses not to provide
>
> — [myrrlyn, July 21, 2021](https://twitter.com/myrrlyn/status/1417804612921344003)

I am proud to say that, thanks to Melanie Blower, Aaron Ballman, Tommy Hoffner, and Erich Keane, a lot of implementation, and some other folk, we now have an integer type that does not do whacky promotions spuriously or change its signage (without explicit casting and go-ahead from the programmer). Width may change, but only in response to performing operations such as multiplication on small bit integers with larger ones (it adjusts to the larger one in the operation). So, for example, adding a `_BitInt(22)` with a `_BitInt(4)` will produce a result with `_BitInt(22)`.

This really changes the game for C23 and embedded development, where users would have to persistently and constantly fight with their type system to do math in the width and signage that they want. Users would also have to get into really dirty fights with bitfields, which is one of the most uncared for parts of the C Standard (not because we don't care, but because the existing rules are kind of bonkers and because nobody has wanted to attempt a more general cleanup of the section for a long time now). Bitfields frequently show up in cursed code and cursed parts of [Shafik Yagmour's quizzes](https://twitter.com/shafikyaghmour/status/1431380142970978308). Their promotions are unintuitive, their interaction with common functionality in the standard painful, and their end-uses confusing for even the hardest working of experts. Not to mention that layout control isn't exactly straightforward in C types to begin with!

`_BitInt(N)` changes that completely. In most cases, it does not do automatic integral promotion (unless you're already dealing with C standard types like `char` and performing mixed-arithmetic with a `_BitInt(N)` involved between other built-in integer types). It retains its signage, and can be made unsigned by prefixing the usual `unsigned` to the type name. This makes it suitable for more than just embedded development, of course: 24-bit values can be extracted and worked on intuitively from FLAC and WAV binary data, SHA-256 integer values can be worked on with normal, human-readable mathematical syntax, and code can look more regular while retaining the precise bit properties.



## N2713 - Integer Constant Expressions (and their use for Arrays)

This was a bit of a problem that stemmed from some implementations being too strong. It made the other implementations embarrassed because they had such girthy, strong, and veiny-muscled constant expression parsers. As such, variable declarations like these:

```cpp
int x[(int)+1.0];
```

were considered constant arrays because the parsers by [the Pillar Compilers](https://www.youtube.com/watch?v=XUhVCoTsBaM) was too amazing.

Meme music aside, while it was nice that their constant expression parsers were exceptionally good, unfortunately that declaration should have been considered a Variable Length Array (VLA). While the compiler backend is free to lower it down to being a static array because it's so smart and capable, other far simpler compilers declared it a VLA because the C Standard's constant expression rules are incredibly limited. Seriously: a lot of what is computed at translation / compile time is actually extremely implementation-specific: there's a lot of people relying on a constant folder / constant parser that is way more beefy than the one in the C standard, which can make for a lot of problems when you're working with someone's home-grown C implementation.

So, even if the compiler can turn it into a normal C array on its backend, it has to be considered a VLA, at least for the purposes of consuming the code in the "front end" and conforming to ISO C. (This means, among other things, that you cannot do `= {24}` on the above array as an initializer, because VLAs cannot be initialized with elements. This was where some code was tripped up when it tried to port between compilers, resulting in unnecessary breakage.)



## N2686 - `#warning` directive

Have you ever wanted to issue a diagnostic but let the user keep on compiling the code, because it was more of a "hey, this thing might go wrong, it might not, just letting you know" sort of problem? Did you want to put something in your library that amounted to a "hey, you're doing great, keep it up, love you!" printout in your user's console without ending their compilation, dear reader? Well, look no more, because C23 is going to have a `#warning` directive! Let your users keep on compiling but guide them towards better solutions by adding well-explained deprecation warnings for headers, telling someone that `CHAR_BIT != 8` is not really supported but "best of luck!!", and all sorts of things that `#error` was just WAY too annoying for!

This feature had been floated around WG21 (C++) a few times and people always got grumpy about it, but here WG14 is leading the way by standardizing existing practice and letting developers talk to their users and give them helpful information (or, perhaps, a well-warranted pat on the back).

Go out there, and put in some helpful diagnostics for your users in C23! 🥳



## N2728 - `char16_t` & `char32_t` String Literals Shall be UTF-16 and UTF-32

One of the things that the entire industry all over has had enough of is `char` and `wchar_t` being implementation-defined shenanigans. This is why everyone who was sick of that nonsense and not trying to port code to compatibility-like environments stuck to `u8`, `u`, and `U` prefixed strings.

Did you know the `char16_t` / `u"aaa"` strings and `char32_t` / `U"aaa"` strings did not have to be UTF-16 or UTF-32 before?

Just like `wchar_t`, these two string literals were not really mandated to be UTF-16 or UTF-32. It was **strongly implied** by everyone hint-hint-wink-wink-nudge-nudge-ing each other, but there was no requirement to do so. An implementation could define `__STDC_UTF_16__` and `__STDC_UTF_32__` macros to be `0` and give you a gigantic middle finger as it filled it with implementation-defined encodings, which they ALSO didn't have to give you a name or identifier for and were not required to document what exactly your `char16_t` or `char32_t` string literals were. Fortunately, the industry all over had been punched in the face enough by `wchar_t` and `L"aaa"` string literals, so **every single implementation on Earth** just picked UTF-16 and UTF-32. I'm serious: we checked over 100 compilers and asked their developers if they ever added or supported a mode where UTF-16 and UTF-32 were not the choices for `u""` and `U""` string literals, respectively. And, uncharacteristically for C implementations, everyone said (if they actually did support it) "no".

We had to take advantage of that.

It's not every day you get people who decide that maybe today is a good day to make a 12-bit `char` machine with a 12-bit text encoding scheme to all agree that UTF-16 and UTF-32 should be the way to go. We initially gave implementations freedom and they did not take it, so we took that freedom back before they decided to shove a rusty spork up every codebase's collective 🍑🍑. And, of course, while we were at it,

we pre-emptively nerd-sniped [these folk](https://reviews.llvm.org/D106577). 🎯

1 month in advance of that Pull Request/Review, this proposal formed the specification that blew up the tying-together of the encoding of string literals `"aaa"` and `L"aaa"` to the locale-dependent `mbstowcs` and `wcstombs`. This is because we knew that, ultimately, people were wasting time debating extraordinarily fanciful scenarios such as "if `__STDC_ISO_10646__` is dependent on the encoding of `L"aaa"`, and `L"aaa"` has wording that says it relies on `mbstowcs`, then we can end up in a situation where the string literal does not match the locale-based wide string encoding on a platform; therefore, it is strictly conforming to just never define the macro even if we encode our wide string literals `L"aaa"` as UTF-32 all the time".

Even if someone wanted to make that argument, it is literally impossible to uphold the wording in the C Standard as it exists. What does `mbstowcs` mean at compile-time? Does this require that I have a conforming C implementation when I make my compiler? What if I'm building my compiler out of assembler and there's nothing else in existence but my goddamn bootloader and a minimal OS, also created out of painstakingly hand-crafted artisanal instructions force-fed directly into my CPU? It is not an interesting or useful question to ask, and the requirement is not a useful one. The only thing that wording needed to be was destroyed, not debated.

It is important to recognize when the rules or letter of the law are not useful. Conformance for conformance's sake is a waste of everyone's time. And it is important, after recognizing that the rules are bad or will lead to a problem, to change them or create exceptions. For some places in the C Standard, we were far too late and it required us to make things that [should never have been undefined behavior into UB to satisfy a broken landscape](http://www.open-std.org/jtc1/sc22/wg14/www/docs/n2464.pdf) where nobody is willing to patch and some people go full Joker to hold the standard at their implementation's gunpoint.

String literals and string processing with `u8""`, `u""`, and `U""` will not be one of them; this proposal was accepted, thank all of Heaven.



## N2799 - `__has_include`

What can I say about this proposal except:

finally.

It's a shame that, for a lot of really good preprocessor features, C++ is getting them first (e.g. `__VA_OPT__` and `__has_include`) since it's C folk that depend on the preprocessor the most. But, that's alright, because a lot of these are also on the docket for C23, and one of them has made it in! Fantastically, `__has_include` is now part of C23, just like it's been a part of C++ for a good while now. It lets us check if a header file exists, which is not a perfect thing to do in C (and very discouraged with standard headers in C++ because the header can exist but the content can be excluded or wrong for the Standard Version you are using), but immensely helpful! A lot of those "checking sys/stat.h exists..." checks in `./configure` are about to become very, **very** irrelevant very quickly, and I am excited for it.

It won't obsolete this stuff but it does mean we are getting closer and closer to writing robust, standalone libraries in C that don't require a mountain of pre-work to get started on many completely normal systems.




# And more...!

There were a lot of other smaller fixes that were accepted to C23, and a few big ones I'm not exactly the most experienced in to talk about. For example, I think David Goldblatt will probably have some words (somewhere) about [N2801](http://www.open-std.org/jtc1/sc22/wg14/www/docs/n2801.htm), which is some exciting work on sized memory deallocation and other fun stuff we could use to increase performance AND safety in the standard with respect to allocation. I'll let him (if he chooses to!) handle that one, even if I think it's super important stuff! [Hans Boehm is also pushing changes to help synchronize `_Atomic(T)` between C and C++](http://www.open-std.org/jtc1/sc22/wg14/www/docs/n2741.htm), which should help close the gap between there and maybe push some implementations to have less of a schism between their C and C++ versions of these types.

We are going to make C23 **the** definitive version of C to upgrade to. Work on `typeof`, forward-declaring parameters, [specifying the underlying type of an enumeration](http://www.open-std.org/jtc1/sc22/wg14/www/docs/n2575.pdf), better [escape sequences](http:/s/www.open-std.org/jtc1/sc22/wg14/www/docs/n2785.pdf), getting rid of [footguns like parameter-less/takes-any-argument functions](https://twitter.com/Cor3ntin/status/1432828206344716288), and other things are coming along. Producing a safer, better, and more programmer-friendly C Standard which rewards your hard work with a language that can meet your needs without 100 compiler-specific extensions and subtle hacks that break is my top priority. One proposal at a time, in the spirit of C. It won't be anything like C++ or Rust or Zig or any of that, but we hope it'll be good for you, dear reader.

Finally, I got (very small, but still positive) approval to go start heckling implementations about one of my Big Bombs™ for shaking up the ABI problem space for C. It likely won't make it for C23, which means your `intmax_t` is gonna stay `long long` for a little bit `long long`er. But that...

is something I'll tell you about a little later.~

Take care, out there! 💚

<sub>— [Title Photo](https://www.pexels.com/photo/brown-cat-with-green-eyes-617278/) by [Kelvin Valerio](https://www.pexels.com/@kelvin809), from Pexels.</sub>