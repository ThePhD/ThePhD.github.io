---
layout: post
title: "Improving _Generic in C2y"
permalink: /improving-_generic-in-c2y
feature-img: "/assets/img/2024/07/stocks-banner.jpg"
thumbnail: "/assets/img/2024/07/stocks-banner.jpg"
tags: [C, C standard, 📜, _Generic]
excerpt_separator: <!--more-->
---

The first two meetings of C after C23 was finalized are over, and we have started working on C2y. We decided that this cycle we're not going to do that "Bugfix" followed by "Release" stuff, because that proved to be a REALLY bad idea that killed a ton of momentum and active contributors during the C11 to C17 timeframe. So, this time, we're hitting both bugfixes AND features so we can make sure we don't lose valuable contributions and fixes by stalling for 5 to 6 years again. So, with that...<!--more--> on to fixes!




# Generic Selection, a Primer

`_Generic` --- the keyword that's used for a feature that is Generic Selection --- is a deeply hated C feature that everyone likes to dunk on for both being too much and also not being good enough at the same time. It was introduced during C11, and the way it works is simple: you pass in an expression, and it figures out the type of that expression and allows you to match on that type. With each match, you can insert an expression that will be executed thereby giving you the ability to effectively have "type-based behavior". It looks like this:

```cpp
int f () {
	return 45;
}

int main () {
	const int a = 1;
	return _Generic(a,
		int: a + 2,
		default: f() + 4
	);
}
```

As demonstrated by the snippet above, `_Generic(...)` is considered an expression itself. So it can be used anywhere an expression can be used, which is useful for macros (which was its primary reason for being). The feature was cooked up in C11 and was based off of [a GCC built-in](https://www.open-std.org/JTC1/SC22/WG14/www/docs/n1404.htm) (`__builtin_choose_expr`) and [an EDG special feature](https://www.open-std.org/jtc1/sc22/wg14/www/docs/n1340.htm) (`__generic`) available at the time, after a few papers came in that said type-generic macros were absolutely unimplementable. While C has a colloquial rule that the C standard library can "require magic not possible by normal users", it was exceedingly frustrating to implement type-generic macros --- specifically, `<tgmath.h>` --- without any language support at all. Thus, `_Generic` was created and a language hole was patched out.

There are, however, 2 distinct problems with `_Generic` as it exists at the moment.



## Problem 0: "l-value conversion"

One of the things the expression put into a `_Generic` expression undergoes is something called "l-value conversion" for determining the type. "l-value conversion" is a fancy "phrase of power" (POP) in the standard that means a bunch of things, but the two things we're primarily concerned about are:

- arrays turn into pointers;
- and, qualifiers are stripped off.

This makes some degree of sense. After all, if we took the example above:

```cpp
int f () {
	return 45;
}

int main () {
	const int a = 1;
	return _Generic(a,
		int: a + 2,
		default: f() + 4
	);
}
```

and said that this example returns `49` (i.e., that it takes the `default:` branch here because the `int:` branch doesn't match), a lot of people would be mad. This helps `_Generic` resolve to types without needing to write something very, very convoluted and painful like so:

```cpp
int f () {
	return 45;
}

int main () {
	const int a = 1;
	return _Generic(a,
		int: a + 2,
		const int: a + 2,
		volatile int: a + 2,
		const volatile int: a + 2,
		default: f() + 4
	);
}
```

In this way, the POP "l-value conversion" is very useful. But, it becomes harder: if you want to actually check if something is `const` or if it has a specific type, you have to make a pointer out of it and make the expression a pointer. Consider this `TYPE_MATCHES_EXPR` bit, Version Draft 0:

```cpp
#define TYPE_MATCHES_EXPR(DESIRED_TYPE, ...) \
	_Generic((__VA_ARGS__),\
		DESIRED_TYPE: 1,\
		default: 0 \
	)
```

If you attempt to use it, it will actually just straight up fail due to l-value conversion:

```cpp
static const int a;
static_assert(TYPE_MATCHES_EXPR(const int, a), "AAAAUGH!"); // fails with "AAAAUGH!"
```

We can use a trick of hiding the qualifiers we want behind a pointer to prevent "top-level" qualifiers from being stripped off the expression:

```cpp
#define TYPE_MATCHES_EXPR(DESIRED_TYPE, ...) \
	_Generic(&(__VA_ARGS__),\
		DESIRED_TYPE*: 1,\
		default: 0\
	)
```

And this will work in the first line below, but FAIL for the second line!

```cpp
static const int a;
static_assert(TYPE_MATCHES_EXPR(const int, a), "AAAAUGH!"); // works, nice!
static_assert(TYPE_MATCHES_EXPR(int, 54), "AAAAUGH!"); // fails with "AAAAUGH!"
```

In order to combat this problem, you can use `typeof` (standardized in C23) to add a little spice by creating a null pointer expression:

```cpp
#define TYPE_MATCHES_EXPR(DESIRED_TYPE, ...) \
	_Generic((typeof((__VA_ARGS__))*)0,\
		DESIRED_TYPE*: 1,\
		default: 0\
	)
```

Now it'll work:

```cpp
static const int a;
static_assert(TYPE_MATCHES_EXPR(const int, a), "AAAAUGH!"); // works, nice!
static_assert(TYPE_MATCHES_EXPR(int, 54), "AAAAUGH!"); // works, yay!
```

But, in all reality, this sort of "make a null pointer expression!!" nonsense is esoteric, weird, and kind of ridiculous to learn. We never had `typeof` when `_Generic` was standardized so the next problem just happened as a natural consequence of "standardize exactly what you need to solve the problem".



## Problem 1: Expressions Only?!

The whole reason we need to form a pointer to the `DESIRED_TYPE` we want is to (a) avoid the consequences of l-value conversion and (b) have something that is guaranteed (more or less) to not cause any problems when we evaluate it. Asides from terrible issues with Variably-Modified Types/Variable-Length Arrays and all of the Deeply Problematic issues that come from being able to use side-effectful functions/expressions as part of types in C (even if `_Generic` guarantees it won't evaluate the selection expression), this means forming a null pointer to something is the LEAST problematic way we can handle any given incoming expression with `typeof`.

More generally, however, this was expected to just solve the problem of "make type-generic macros in C to implement `<tgmath.h>`". There was no other benefit, even if a whole arena of cool uses grew out of `_Generic` and its capabilities (including very very basic type inspection / queries at compile-time). The input to type-generic macros was always an expression, and so `_Generic` only needed to take an expression to get started. There was also no standardized `typeof`, so there was no way to take the `INPUT` parameter or `__VA_ARG__` parameter of a macro and get a type out of it in standard C anyways. So, it only seemed natural that `_Generic` took only an expression. Naturally, as brains got thinking about things,

someone figured out that maybe we can do a lot better than that!




# Moving the Needle

Implementers had, at the time, been complaining about not having a way to match on types directly without doing the silly pointer tricks above because they wanted to implement tests. And some of them complained that the standard wasn't giving them the functionality to solve the problem, and that it was annoying to reinvent such tricks from first principles. This, of course, is at the same time that implementers were **also** saying we shouldn't just bring papers directly to the standard, accusing paper authors of "inventing new stuff and not standardizing existing practice". This, of course, did not seem to apply to their own issues and problems, for which they were happy to blame ISO C for not figuring out a beautiful set of features that could readily solve the problems they were facing.

But, one implementer then got a brilliant idea. What if they flexed their implementer muscles? What if **they** improved `_Generic` and reported on that experience without waiting for C standard to do it first? What if implementers fulfilled their end of the so-called "bargain" where they actually implemented extensions? And then, as C's previous charters kept trying to promise (and then fail to deliver on over and over again over decades), what if those implementers then turned around to the C standard to standardize their successful existing practice so that we could all be Charter-Legal about all of this? After all, it would be way, WAY better than being perpetually frozen with fear that if they implemented a (crappy) extension they'd be stuck with it forever, right? It seems like a novel idea in this day and age where everything related to C seems conservative and stiff and boring. But?

Aaron Ballman decided to flex those implementer muscles, bucking the cognitive dissonance of complaining that ISO C wasn't doing anything, not writing a paper, and not follow up on his own implementation. He kicked off the discussion. He pushed through with the feature. And you wouldn't believe it, but:

it worked out great.




# N3260 - Generic Selection Expression with Type Operand

It's as simple as the paper title: [N3260 puts a type where the expression usually goes](https://www.open-std.org/JTC1/SC22/WG14/www/docs/n3260.pdf). Aaron got it into Clang in a few months, since it was such a simple paper and had relatively small wording changes. Using a type name rather than an expression in there, `_Generic` received the additional power to get **direct matching** with no l-value conversion. This meant that qualifier stripping --- and more -- did not happen. So we can now write `TYPE_MATCHES_EXPR` like so:

```cpp
#define TYPE_MATCHES_EXPR(DESIRED_TYPE, ...) \
	_Generic(typeof((__VA_ARGS__)),\
		DESIRED_TYPE: 1,\
		default: 0\
	)

static const int a;
static_assert(TYPE_MATCHES_EXPR(const int, a), "AAAAUGH!"); // works, nice!
static_assert(TYPE_MATCHES_EXPR(int, 54), "AAAAUGH!"); // works, nice!
```

This code looks normal. Reads normal. Has no pointer shenanigans, no null pointer constant casting; none of that crap is included. You match on a type, you check for exactly that type, and life is good.

Clang shipped this quietly after some discussion and enabled it just about everywhere. GCC soon did the same in its trunk, because it was just a good idea. Using the flag `-pedantic` will have it be annoying about the fact that it's a "C2y extension" if you aren't using the latest standard flag, but this is C. You should be using the latest standard flag, the standard has barely changed in any appreciable way in years; the risk is minimal. And now, the feature is in C2y officially, because Aaron Ballman was willing to kick the traditional implementer Catch-22 in the face and be brave.

Thank you, Aaron!

The other compilers are probably not going to catch up for a bit, but now `_Generic` is much easier to handle on the two major implementations. It's more or less a net win! Though, it... DOES provide for a bit of confusion when used in certain scenarios, however. For example, using the same code from the beginning of the article, this:

```cpp
int f () {
	return 45;
}

int main () {
	const int a = 1;
	return _Generic(typeof(a),
		int: a + 2,
		default: f() + 4
	);
}
```

does not match on `int` anymore, IF you use the type-based match. In fact, it will match on `default:` now and consequently will call `f()` and add `4` to it to return `49`. That's gonna fuck some people's brains up, and it will also expose some people to the interesting quirks and flaws about whether certain expressions --- casts, member accesses, accesses into qualified arrays, etc. --- result in specific types. We've already uncovered one fun issue in the C standard about whether this:

```cpp
struct x { const int i; };

x f();

int main () {
	return _Generic(typeof(f().i),
		int: 1,
		const int: 2,
		default: 0
	);
}
```

will make the program return `1` or `2` (the correct answer is `2`, but GCC and Clang disagree because of course they do). More work will need to be done to make this less silly, and I have some papers I'm writing to make this situation better by tweaking `_Generic`. `_Generic`, in general, still needs a few overhauls so it works better with the compatibility rules and also doesn't introduce very silly undefined behavior with respect to Variable-Length Arrays and Fixed-Size Array generic types. But that's a topic

for another time. 💚

{% include anchors.html %}
