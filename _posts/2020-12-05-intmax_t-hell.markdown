---
layout: post
title: A Special Kind of Hell - intmax_t in C and C++
permalink: /intmax_t-hell-c++-c
feature-img: "/assets/img/2020/12/burning-book.jpg"
thumbnail: "/assets/img/2020/12/burning-book.jpg"
tags: [ABI, C++, C, Standard, 📜, ⌨]
excerpt_separator: <!--more-->
---

C and C++ as languages have a few things separating them from each other, mostly in their minute details and, occasionally, larger feature sets like designated initializers. But there is a disturbingly high amount of C++ that can simply do C's job far better than C<!--more-->, including when it comes to solving some of the biggest problems facing the evolution of C and C++.

Let's take a contemporary problem plaguing both C and C++, affecting everyone from standard library maintainers to project developers, that has been going on for the last 20 or so years: `intmax_t`.




# Maximum Integer Types

The concept behind `intmax_t` is simple enough: it is the largest integer type that your implementation and its standard library support in conjunction. Here is a few things `intmax_t` controls inside the implementation:

- numeric literals are preprocessed according to what `intmax_t` can handle (C and C++);
- it is the maximum number of bits that can be printed portably, e.g. with `printf("%j", (intmax_t)value)` (C and C++);
- `intmax_t` is the largest type for which `std::numeric_limits` applies, including most types up to and including that type (C++ only);
- `intmax_t` underpins `std::chrono`'s casts and similar (e.g. no information is lost during conversions out of and into the system) (C++ only);
- and, there are a set of integer operations provided by the standard library (like absolute value and quotient / remainder operations) that can be done with the maximum bit precision available to the implementation (C and C++).

These properties forge the basis of `intmax_t`'s purpose. Lossless storage, pass-through operations, and more can all be achieved by relying on this implicit contract of the type. Since it is a type definition, the "real" integer type underneath it can be swapped out and people relying on it can be upgraded seamlessly!




# Just Kidding

We cannot upgrade seamlessly.

C has a much higher commitment to not breaking old code and keeping "developers close to the machine". What this actually translates to for most Application Binary Interfaces is very simplistic "name mangling" schemes (i.e., none), [weak linkers](https://web.archive.org/web/20201121013636/https://twitter.com/__phantomderp/status/1329960075096694790), and other shenanigans. The end result is that we expose C developers to platform details that become invisible dependencies for their code that must be preserved at all costs. For example, let's take a C Standard function that uses `intmax_t`, `imaxabs`:

```cpp
intmax_t imaxabs(intmax_t j);
```

and, let's try to figure out how we can upgrade someone off of this usage without breaking their code too badly. We will try fixing this in both C and C++.




# A Demonstration

Taking `intmax_t`, let's do a no-brainer usage case: calling a function with `intmax_t` input and return types. The syntax and usage ends up looking like this:

```c
#include <inttypes.h>

int main () {
	intmax_t original = (intmax_t)-2;
	intmax_t val = imaxabs(original);
	return (int)val;
}
```

Easy enough! But, there's also a hidden dependency here, based on how the code is compiled. While many people compile their C standard library as a static library and only generate final binary code for what they use as to have a "self-contained" binary, the vast majority of the shared ecosystem depends on shared libraries/dynamically linked libraries for the standard. This means that when a program is milled through an operating system at program startup, the "loader" runs off to find the symbol `imaxabs` inside some system library (e.g., `/lib/x86_64-linux-gnu/libc-2.27.so` for an "amd64" system). Harmless enough, right? Well, it turns out to be a bit of a problem in practice, because the name `imaxabs` is all that's used in C to figure out what subroutine to talk with in some shared library,

and that name is completely inadequate.

Consider the following scenario:

0. The glibc maintainers decide they're going to change from `long long` as their `intmax_t` and move to `__int256_t` for most platforms, because most platforms support it and they have a lot of customers asking for it.
1. They upgrade the `libc` to its next version for various Linux distribution, and everyone links against it when they look for the default `libc`.
2. You have an application. Your code was not changed or updated, so it was not recompiled. It calls `imaxabs`. The argument it passes is a `long long`, because that was the type at the time you last compiled and shipped your software.
3. The `imaxabs` used to lookup the function to call finds the version that takes a `__int256_t` in the new `libc`.
4. Different registers are used to pass and return the function value than expected by the `imaxabs` function call in the `libc` binary, because your application is in `long long` mode but glibc expects a `__int256_t`.
5. All hell breaks loose.

This is one of the manifestations of what is called an "Application Binary Interface (ABI) Break". ABI Breaks are generally undetectable, silent breaks that occur within the runtime of a program that completely destroy any dependency your program has on that functionality for correctness. It typically happens when a subtle detail -- the registers used to negotiate a large integral value between a shared library and its application, the amount of padding a structure might have on a certain build, the ordering and layout of class members, the interpretation of bits even if the layout or passing convention of a type never changes, and even more -- changes.



## "But C Is ABI-Stable?!"

Not necessarily. C is a simple language, and it both sells itself on and prides itself as such. So much so, that it's even part of the [language's rolling charter](https://www.open-std.org/jtc1/sc22/wg14/www/docs/n2021.htm). There's barely any name mangling because there's no overloading. If you want "virtual functions" you need to hand-craft your virtual table structure and initialize it yourself. There's barely any lookup or entity negotiation: what you write -- [however scary or cursed](https://twitter.com/thingskatedid/status/1328918322507706368) -- is what you get, in a general sense. (No, it's not "portable assembly". Compilers tear C code apart and make it far more efficient than the code people stuff into it. It's not even a direct model of the machine anymore: just an abstract one.)

Still, sometimes even C can't get away from it. The function `imaxabs` relates to exactly one entity that, for historical reasons, was pinned to a function taking and returning a `long long`. Upgrading it means dealing with this schism between what the user expects (`intmax_t` that got upgraded and can print `__int128_t`/`__int256_t`) with old, non-recompiled code that maintains the old invariant (`long long`, a 64-bit number).




# Mitigation Strategies

Okay, so symbols can be repurposed between library versions that lead to ABI breaks. What are the ways to defend against such a world, in C?



## Macros?

Macros! Object-like macros are fun. You could do something like this...

```c
#define imaxabs __glibc228_imaxabs
```

... as a way to provide the `imaxabs` function. It is a bit like artisanal, hand-crafted, free-range, and organic ABI versioning (or, as I have affectionately come to call it: personal masochism to make up for language failures). This mostly works, until... it doesn't!



### §7.1.4

This is the "ABI Breaks Guaranteed" section in the C Standard. It's real name is "§7.1.4 Use of library functions". Reproduced below is the relevant piece that condemns us, emphasis mine:

> Any function declared in a header may be additionally implemented as a function-like macro defined in the header, so if a library function is declared explicitly when its header is included, one of the techniques shown below can be used to ensure the declaration is not affected by such a macro. **Any macro definition of a function can be suppressed locally by enclosing the name of the function in parentheses**, because the name is then not followed by the left parenthesis that indicates expansion of a macro function name. For the same syntactic reason, it is permitted to take the address of a library function even if it is also defined as a macro. **The use of `#undef` to remove any macro definition will also ensure that an actual function is referred to**.

Not only can a user suppress a function-like macro invocation by using the same trick used on `<windows.h>` like `(max)(value0, value1)`, but the C Standard Library permits them to also undefine function names:

```c
// implementer code: inttypes.h
#define imaxabs __glibc228_imaxabs
```

```c
// user code: main.c
#include <inttypes.h>

#undef imaxabs // awh geez

int main () {
	intmax_t val = -1;
	intmax_t absval = imaxabs(val); // awH GEEZ
	return (int)absval;
}
```

Mmm......



### Implementation-specific strategies

Alright, the C Standard basically loads a double barrel and brings our only standardized mitigation strategy out back behind the barn. What's left? Well, implementation-specific insanity, that's what:

```c
extern
intmax_t
__glibc228_imaxabs(intmax_t);

__attribute((symbol(__MANGLE(__glibc228_imaxabs))))
extern
intmax_t
imaxabs(intmax_t);
```

This is pseudo-code. But, wouldn't you believe it, some implementations actually do things very similar to this to get around these problems! The things they do are far more involved, like actually dropping down to the level of the linker and creating symbol maps and other exceedingly painful workarounds. The sed scripts and the awk scripts and the bash starts coming out, people are doing lots of text processing to get symbol names and match them to versioned symbol names...

It's a mess.

Still, given the mess, it does save us from the problem. In C code you get to use "the real name" `imaxabs` as Our Lord and Savior intended, the binary gets linked to `___glibc228_imaxabs`, and everyone's happy. There's only one problem with this kind of fix...

It's Quality of Implementation (QoI).

QoI is great for the pure, theoretical standard. We get to write sexy narratives in the C standard and call them "Recommended Practice", with little footnotes furtively implying a more wonderful world while waggling our eyebrows seductively at hot, young developers in our area. Just come along, it's going to be so great, we're going have soooooo much fun, just go with that lovely little implementation right over there, you're making such fine progress, enjoy yourself and come back soon my purrecious Software Engineer!~

Then you wake up in an alleyway with your bit wallet pillaged and your preconditions violated. Your head is swimming from unspecified behavior, and it turns out that the implementation:

- did not do any quality of implementation;
- splat out the symbols in the global namespace;
- learned 0 lessons from `size_t` requiring the entire industry over to have different binaries for 64-bit just to change its definition;
- dumped out a plain old `extern` function definition;

Hurt, confused, you come to us, the Standards Committee. You tell us that you're bamboozled and bruised, that this wonderful implementation named [TenDRA](http://www.tendra.org/) said you could do these amazing things, that you'd get to see standards-conforming explosions and fireworks, that it would be rad...

And we just sigh.

We explain that, see, it's not illegal, what TenDRA did. No, you don't get automatic access to their exclusive footnotes. We have to serve the implementations that don't actually want to do the recommended practice, why didn't you check your implementation's documentation, how come you didn't contact the maintainers? Really there's nothing we can do for you here, you had the freedom to pick, why didn't you pick a better implementation? If your ideas were really so popular, how come they haven't been implemented the way you expected, huh? Why are you not pushing TenDRA to the left—

As a governing body WG14 will take a recommended implementation seriously. We will also take the DeathStation 9000 seriously. It's conforming, after all. This isn't the implementation's fault:

it's C's fault.




# The Monster in the Mirror

In our desperate bid to hammer out rules for ourselves and make sure all code ever written in C forever continues to compile forever, we tied our hands as a Standards Committee. We get frustrated with ABI problems with `intmax_t` and in `printf`, but the honest to god truth is that we asked for this.

We cornered ourselves, with painstaking deliberation and intent.

The [bug reports roll in](https://gcc.gnu.org/bugzilla/show_bug.cgi?id=96710) to C and C++ implementations, the [quirky patches get started](https://gcc.gnu.org/pipermail/libstdc++/2020-August/050792.html). [Why is `__int128_t` a pariah](https://quuxplusone.github.io/blog/2019/02/28/is-int128-integral/), where's the up-and-coming `__int256_t` support, why can't we use `__int512_t` with `numeric_limits`, why is `_ExtInt(1024)` not a "real extended integer type", et cetera et cetera. We would like to blame ABI but the fault is our own at this point: we crafted a world where Names Are Forever, Abstractions Are Evil, Macros Are Bad, and now the rules we made to chain the complexity of the world of C are now staring us back in the face with one hell of a wicked grin. We valued simplicity to the point of self-parody and -- to some extent -- self-destruction.



## The Fix

For C? None. I am not even kidding: no less than [5 papers](https://www.open-std.org/jtc1/sc22/wg14/www/docs/n2465.pdf) discussed over [3-4 Standards meetings](https://www.open-std.org/jtc1/sc22/wg14/www/docs/n2498.pdf) have been [burned on this topic](https://www.open-std.org/jtc1/sc22/wg14/www/docs/n2425.pdf). There's no combination of ISO Standard C features that can fix this problem without breaking somebody. Not even the upcoming idea for [lambdas and auto (see §6.5.2.6 Lambda expressions)](https://www.open-std.org/jtc1/sc22/wg14/www/docs/n2522.pdf) in C can fix this problem:

```c
extern inline
auto
imaxabs = [](intmax_t value) -> intmax_t {
	return __actual_imaxabs(value);
};
```

On the surface, this seems like it would work, until this happens:

```c
int main () {
	typedef (intmax_t)(f_t)(intmax_t);
	f_t* f_ptr0 = imaxabs; // fine, function pointer decay
	f_t* f_ptr1 = &imaxabs; // oops
	return 0;
}
```

By how lambdas are defined, that returns an object pointer, not a function pointer. This means old code using this syntax is broken. It also brings into question what, exactly, _would_ be the address of that function pointer? A feature would need to be crafted as "lambdas, but you can control what the 'address of' operator returns". Of course, just writing that sentence has ten thousand C developers crying out: "No, C must be simple, operator overloading is the DEVIL'S MISTRESS!".

Which, speaking of the Devil's Mistress...



## The Fix, in C++

C++ can solve their ABI issue according to their own library rules, today. We just need to tap into a little bit of that spicy devilry:

```cpp
namespace std {

	extern "C" __int128_t __i128abs (__int128_t v2) noexcept;

	using intmax_t = __int128_t;

	struct __imaxabs {
	private:
		using __imaxabs_ptr = intmax_t(*)(intmax_t) noexcept;

	public:
		constexpr __imaxabs_ptr operator& () const noexcept {
			return &__i128abs;
		}

		constexpr operator __imaxabs_ptr () const noexcept {
			return &__i128abs;
		}
	};

	inline constexpr const __imaxabs imaxabs = __imaxabs{};
}
```

Unfortunately, we tied C++ itself to C compatibility. That means we will never take a step like this, because that means the C Library's definition of `intmax_t` on your system would differ from your C++ library. Ensue shenanigans, etc. etc. Note that it's not impossible for C++ to do this even still, because of this wording:

> The C++ standard library also makes available the facilities of the C standard library, suitably adjusted to ensure static type safety.
> — [C++ Working Draft, §16.2 The C standard library ([library.c/1])](http://eel.is/c++draft/library.c#1)

Much as we could use the above to weasel our way out of staying identical, implementations want so badly for `imaxabs(value)` to have the exact same meaning as `std::imaxabs(value)`. So, yes, maybe it's true that...




# C++ is a better C

But we explicitly choose for it not to be.

When it comes to covering the same feature set as C, we specifically will not push the envelope or go beyond our limitations. We will load our footgun proudly and blow the leg off of our fundamentals, then [force well-meaning developers](https://twitter.com/stephentyrone/status/1329796144193556482) into a implementation-defined hell fire. Compatibility is a feature and it is absolutely worth it, of course!

Of course.

The only way to fix this in reality is defining your own C library implementation that had aggressive symbol versioning support from the implementation and an investment in forward compatibility. And honestly, who's wild enough to do that in this day and age?

[Nobody, obviously...](https://www.reddit.com/r/cpp/comments/jqxan9/a_libc_written_in_c/)

Obviously. 💚

— ThePhD



<sub>Modified Title Photo by [Joy Marino](https://www.pexels.com/@joymarino?utm_content=attributionCopyText&utm_medium=referral&utm_source=pexels) from [Pexels](https://www.pexels.com/photo/woman-wears-black-sleeveless-top-3052651/?utm_content=attributionCopyText&utm_medium=referral&utm_source=pexels)</sub>

{% include anchors.html %}
