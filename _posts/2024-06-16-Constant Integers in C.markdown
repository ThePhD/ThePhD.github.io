---
layout: post
title: "Constant Integer Type Declarations Initilized With Constant Expressions Should Be Constants"
permalink: /constant-integers-are-actually-constant-wow-finally-someones-writing-the-goddamn-paper-🙄
feature-img: "/assets/img/2024/06/equations-banner.jpg"
thumbnail: "/assets/img/2024/06/equations-banner.jpg"
tags: [C, ICE, Integer Constant Expressions, Constant Expressions, C standard, 📜, Finally]
excerpt_separator: <!--more-->
---

Constant integer-typed (including enumeration-typed) object declarations in C that are immediately initialized with an integer constant expression should just be constant expressions. That's it.<!--more--> That's the whole article; it's going to be one big propaganda piece for an upcoming change I would **like** to make to the C standard for C2y/C3a!



# Doing The "Obvious", Obviously

As per usual, everyone loves complaining about the status quo and then not doing anything about it. Complaining is a fine form of feedback, but the problem with a constant stream of crticism/feedback is that nominally it has to be directed — eventually — into some kind of material change for the better. Otherwise, it's just a good way to waste time and burn yourself out! As one would correctly imagine, this "duh, this is obvious" feature is not in the C standard. But, it seemed like making this change would take too much time, effort, and would be too onerous to wrangle. However, this is no longer the case anymore!

Thanks to changes made in C23 by Eris Celeste and Jens Gustedt (woo, thanks you two!), we can now write a very simple and easy specification for this that makes it terrifyingly simple to accomplish. We also know this will not be an (extra) implementation burden to conforming C23 compilers for the next revision of the standard thanks to `constexpr` being allowed in C23 for object declarations (but not functions!). As we now have such `constexpr` machinery for objects, there is no need to go the C++ route of trying to accomplish this in the before-`constexpr` times. This makes both the wording and the semantics easy to write about and reason about.




# How It Works

The simple way to achieve this is to take every non-`extern`, `const`-qualified (with no other qualifiers) integer-typed (including `enum`-typed) declaration and upgrade it implicitly to be a `constexpr` declaration. It only works if you're initializing it with an integer constant expression (a specific kind of Phrase of Power in C standardese), as well as a few other constraints. There are a few reasons for it to be limited to `non`-extern declarations, and a few reasons for it to be limited to integer and integer-like types rather than the full gamut of floating/`struct`/`union`/etc. types. Let's take a peak into some of the constraints and reasonings, and why it ended up this way.



## Integer Types? Why Not "Literally Everything™"?

Doing this for integer types is more of a practicality than a full-on necessity. The reason it is practical is because 99% of all compilers already compute integer constant expressions for the purposes of the preprocessor and the purposes of the most basic internal compiler improvements. Any serious commercial compiler (and most toy compilers) can compute `1 + 1` at compile-time, and not offload that expression off to a run-time calculation.

However, we know that most C compilers do not go as far as GCC or Clang which will do its damnedest to compute not only every integer constant expression, but compound literal and structure initialization expression and string/table access at compile-time. If we extend this paper to types beyond integers, then we quickly exit the general blessing we obtain from "We Are Standardizing Widely-Deployed Existing Practice". At that point, we would not be standardizing widespread existing practice, but instead the behavior of a select few powerful compilers whose built-in constant folders and optimizers are powerhouses among the industry and the flagships of their name.

C++ does process almost everything it can at compile-time when possible, under the "manifestly constant evaluated" rules and all of its derivatives. This has resulted in serious work on the forward progress of constant expression parsers, including a whole new constant expression interpreter in Clang[^clang-interpreter]. However, C is not really that much of a brave language; historically, standard and implementation-provided C has been at least a decade (or a few decades) behind what could be considered basic functionality, requiring an independent hackup of what are bogstandard basic features from users and vendors alike. Given my role as "primary agitator for the destruction of C" (or improvement of C; depends on [who's being asked at the time](https://gavinhoward.com/2024/05/a-grateful-open-letter-to-jeanheyd-meneide/)), it seems fitting to take yet another decades-old idea and try to get it through the ol' Standards Committee Gauntlet.

With that being the case, the changes to C23's constant expression rules were already seen as potentially harmful for smaller implementations. (Personally, I think we went exactly as far as we needed to in order to make the situation less objectively awful.) So, trying to make ALL initializers be parsed for potential constant expressions would likely be a bridge too far and ultimately tank the paper and halt any progress. Plus, it turns out we tried to do the opposite of what I'm proposing here! And,

it actually got dunked on by C implementers?!



## We Failed To Do It The Opposite Way

A while back, I wrote about the [paper N2713](/c-the-improvements-june-september-virtual-c-meeting#n2713---integer-constant-expressions-and-their-use-for-arrays) and how it downgraded implementation-defined integer constant expressions to be treated like normal numbers "for the purposes of treatment by the language and its various frontends". This was a conservative fix because, as the very short paper stated, there was implementation divergence and smaller compilers were not keeping up with the larger ones. Floating point-to-integer conversions being treated as constants, more complex expressions, even something like `__builtin_popcount(…)` function calls with constants being treated as a constant expression by GCC and Clang were embarrassing the smaller commercial offerings and their constant expression parsers.

It turns out that implementation divergence mattered a **lot**. A competing paper got published during the "fix all the bugs before C23" timeframe, and it pointed all of this out in [paper N3138 "Rebuttal to N2713"](https://www.open-std.org/jtc1/sc22/wg14/www/docs/n3138.pdf). The abstract of N3138 makes it pretty clear: "[N2713] diverges from existing practice and breaks code." While we swear up and down that existing implementations are less important in our Charter (lol), the Committee DOES promise that existing code in C (and sometimes, C-derivative) languages will be protected and priotized as highly as is possible. This ultimately destroyed N2713, and resulted in it being considered implementation-defined again whether or not non-standards-blessed constant expressions could be considered constants.

Effectively, the world rejected the idea that being downgraded and needing to ignore warnings about potential VLAs (that would get upgraded to constant arrays at optimization time) was appropriate. Therefore, if C programmers rejected going in the direction that these had to be treated for compiler frontend purposes as not-constants, we should instead go in the **opposite** direction, and start treating these things as constant expressions. So, rather than downgrading the experience (insofar as making certain expressions be not constants and not letting implementations upgrade them in their front-ends, but only their optimizers), let's try upgrading it!



## Formalizing the Upgrade

In order to do this, I have written a paper [currently colloquially named NXXX1 until I order a proper paper number](/_vendor/future_cxx/papers/C%20-%20Initialized%20const%20Integer%20Declarations.html). The motivation is similar to what's in this blog post, and it contains a table that can explain the changes better than I possibly ever could in text. So, let's take a look:

```cpp
int file_d0 = 1;
_Thread_local int file_d1 = 1;
extern int file_d2;
static int file_d3 = 1;
_Thread_local static int file_d4 = 1;
const int file_d5 = 1;
constexpr int file_d6 = 1;
static const int file_d7 = 1;

int file_d2 = 1;

int main (int argc, char* argv[]) {
	int block_d0 = 1;
	extern int block_d1;
	static int block_d2 = 1;
	_Thread_local static int block_d3 = 1;
	const int block_d4 = 1;
	const int block_d5 = file_d6;
	const int block_d6 = block_d4;
	static const int block_d7 = 1;
	static const int block_d8 = file_d5;
	static const int block_d9 = file_d6;
	constexpr int block_d10 = 1;
	static constexpr int block_d11 = 1;
	int block_d12 = argc;
	const int block_d13 = argc;
	const int block_d14 = block_d0;
	const volatile int block_d15 = 1;

	return 0;
}

int block_d1 = 1;
```

| Declaration |  `constexpr` Before ? |    `constexpr` After ? | Comment                                                     |
|-------------|-----------------------|-----------------------|--------------------------------------------------------------|
| file_d0     | ❌                    | ❌                   | no change; `extern` implicitly, non-`const`                  |
| file_d1     | ❌                    | ❌                   | no change; `_Thread_local`, `extern` implicitly, non-`const` |
| file_d2     | ❌                    | ❌                   | no change; `extern` explicitly, non-`const`                  |
| file_d3     | ❌                    | ❌                   | no change; non-`const`                                       |
| file_d4     | ❌                    | ❌                   | no change; `_Thread_local`, non-`const`                      |
| file_d5     | ❌                    | ❌                   | no change; `extern` implicitly                               |
| file_d6     | ✅                    | ✅                   | no change; `constexpr` explicitly                            |
| file_d7     | ❌                    | ✅                   | `static` and `const`, initialized by constant expression     |
| block_d0    | ❌                    | ❌                   | no change; non-`const`                                       |
| block_d1    | ❌                    | ❌                   | no change; `extern` explicitly, non-`const`                  |
| block_d2    | ❌                    | ❌                   | no change; non-`const`, `static`                             |
| block_d3    | ❌                    | ❌                   | no change; `_Thread_local`, `static`, non-`const`            |
| block_d4    | ❌                    | ✅                   | `const`; initialized with literal                            |
| block_d5    | ❌                    | ✅                   | `const`; initialized with other `constexpr` variable         |
| block_d6    | ❌                    | ✅                   | `const`, initialized by other constant expression            |
| block_d7    | ❌                    | ✅                   | `static` and `const`, initialized with literal               |
| block_d8    | ❌                    | ❌                   | no change; non-constant expression initializer               |
| block_d9    | ❌                    | ✅                   | `static` and `const`, initialized by constant expression     |
| block_d10   | ✅                    | ✅                   | no change; `constexpr` explicitly                            |
| block_d11   | ✅                    | ✅                   | no change; `constexpr` explicitly                            |
| block_d12   | ❌                    | ❌                   | no change; non-`const`, non-constant expression initializer  |
| block_d13   | ❌                    | ❌                   | no change; non-constant expression initializer               |
| block_d14   | ❌                    | ❌                   | no change; non-constant expression initializer               |
| block_d15   | ❌                    | ❌                   | no change; `volatile`                                        |

For the actual "words in the standard" changes, we're effectively just making a small change to ["§6.7 Declarations, §6.7.1 General"](https://www.open-std.org/jtc1/sc22/wg14/www/docs/n3220.pdf#subsection.6.7.1) in the latest C standard. It's an entirely new paragraph that just spins up a bulleted list, saying:

>  <sup>(NEW)13✨</sup> If one of a declaration’s init declarator matches the second form (a declarator followed by an equal sign = and an initializer) meets the following criteria:
> 
> — it is the first visible declaration of the identifier;
> 
> — it contains no other storage-class specifiers except static, auto, or register;
> 
> — it does not declare the identifier with external linkage;
> 
> — its type is an integer type or an enumeration type that is const-qualified but not otherwise qualified, and is non-atomic;
> 
> — and, its initializer is an integer constant expression (6.6);
> 
> then it behaves as if a constexpr storage-class specifier is implicitly added for that declarator specifically. The declared identifier is then a named constant and is valid in all contexts where a named constant of the corresponding type is valid to form a constant expression of that specific kind (6.6).

Thanks to the improvements to [§6.6](https://www.open-std.org/jtc1/sc22/wg14/www/docs/n3220.pdf#section.6.6) from Celeste and Gustedt, and their work on `constexpr`, the change here is very small, simple, and minimal. This covers all the widely-available existing practice we care about, without providing undue burden for many serious C implementations of C23 and beyond. It also would make a wide variety of integer constant expressions from the "Rebuttal" paper [N3138](https://www.open-std.org/jtc1/sc22/wg14/www/docs/n3138.pdf) into valid constant expressions, according to the current rules of the latest C standard. This would be an improvement as it would mean the constant expressions written by users could be **relied** on across platforms that use a `-std=c2y` flag or claim to conform to the latest (working draft) C standard.




# All in All, Though?

I'm just hoping I can get something as simple as this into C. It's been long overdue given the number of ways folks complain about how C++ has this but C doesn't, and it would deeply unify existing practice across implementations. It also helps to remove an annoying style of diagnostic warnings from `-Wpedantic`/`-Wall`-style warning lists, too!

The next meeting for C is around October, 2024. I'll be trying to bring the paper there, to get it formalized, along with the dozens of other papers and features I am working on. Even if my hair will go fully grey by the time this is available on all platforms, I will keep working at it. We deserve the C that people keep talking about, on **all** implementations.

If not in my lifetime, in yours. 💚




[^clang-interpreter]: You can read a writeup about it on RedHat's blog ([Part 1](https://www.redhat.com/en/blog/new-constant-expression-interpreter-clang), [Part 2](https://www.redhat.com/en/blog/new-constant-expression-interpreter-clang-part-2)), or directly [from the LLVM documentation](https://clang.llvm.org/docs/ConstantInterpreter.html).

{% include anchors.html %}
