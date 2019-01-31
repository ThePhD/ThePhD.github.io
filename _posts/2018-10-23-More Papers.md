---
layout: post
title: San Diego Pregame - Paper Review II
permalink: /sandiego-2018-pregame-paper-review-II
redirect_from: [ /2018/10/23/More-Papers.html ]
feature-img: "assets/img/pexels/disco-full.jpeg"
thumbnail: "assets/img/2018-10-23/san-diego.jpg"
tags: [C++, future, standards, proposals, review, ⌨️, 📜]
excerpt_separator: <!--more-->
---


It's time to review a few more papers! Last time, we did simple papers (which still generated a lot of... interesting discussion) but now we are going to move on to some larger, meatier papers.<!--more--> These won't be the largest papers the mailing has to offer, but it's still good to look into them and get some early opinions on them. One of the ones below is not all too great, but starts a necessary conversation. The others are good stabs at solving larger problems that have plagued library developers for years!


### [p1281 - Feature Presentation](https://wg21.link/p1281)

Feature Presentation is the introduction of 2 new things into the standard: an attribute `[[feature(string)]]` that controls whether or not a declaration/definition exists or appears in the eyes of the compiler, and a function `constexpr std::feature(string_view);`. Both pull from the same implementation-defined pool of strings in order to cull declarations/definitions from sight before being processed. The `std::feature` function is also usable in `if constexpr` blocks.

I am not sure I like the direction this paper takes. Attributes that can wipe out whole declarations/definitions are in-theory a fun idea, but being able to use them at the level of per-class and per-function seems like not the right kind of hammer to solve the problem of "compiling fundamentally different code on different platforms". Despite this, p1281 is necessary to start the discussion. Nothing gets done in the C++ Standards Committee without a paper, and whenever someone proposes even a small fix to the language, this response always shows up:

> Write a paper. :)

This is that paper. It doesn't have the form or function I would like yet: attributes just do not seem like a good fit for what this is can do. Albeit, this might also be because I am stuck thinking in "Preprocessor" mode and I do not see the bigger picture of what kind of nice code this may enable.

What I would like to see from this paper is discussion of how this attribute could work at the namespace level. Coupled with inline namespaces, the `[[feature(...)]]` attribute could become a much more powerful form of elevating declarations from a platform or ABI-specific namespaces up into the general namespace depending on the value of the feature attribute. But, despite that potential utility, I would need to see more examples and more demonstrations about how this would work before feeling comfortable supporting it.

I would be either Neutral, or Against.


### [p1249 - forward from initializer_list](https://wg21.link/p1249)

Alex Christensen blesses us with a fix to a long-standing problem everyone has grumbled about repeatedly. There's even a [presentation](https://www.youtube.com/watch?v=sSlmmZMFsXQ) about how cruddy `initializer_list`s are (and it's a great presentation: you should watch it). The proposed change is very simple: strike `const`-ness from `std::initializer_list`'s relevant typedefs and function declarations, making it possible to forward and manipulate the values themselves in a sane manner. To this day, I am still entirely unsure why `initializer_list`'s contents were made to be `const`: it seems some people had a strong inclination that they wanted the array that was going to be used to hold and feed the values to these various constructors from the `initializer_list` would be stored in read-only memory. This makes some sense in that `initializer_list` expressions could be created once and then passed on to be copied many times over whenever the constructor or function in question was called. The relevant section is [§9.3.5, clause 5](http://eel.is/c++draft/dcl.init.list#5), where initializers lists behave "as if":

```c++
struct X {
  X(std::initializer_list<double> v);
};
X x{ 1,2,3 };
```

is like:

```c++
const double __a[3] = {double{1}, double{2}, double{3}};
X x(std::initializer_list<double>(__a, __a+3));
```

By forcing this "array of _N_ `const E`" storage scheme on implementers it becomes impossible to efficiently move created values into constructors (particularly for containers) or functions! (At least, not without dirty and naughty `const_cast`.) And thusly, that is where this paper picks up: by removing the `const` and fixing up clause 5, one could easy gain optimal, non-forced-copy performance. This is especially essential in cases where values in the initializer_list are constructed with runtime parameters, and so it makes zero sense to force `const`, persistent storage for something that needs to always be created on every use: if this is necessary, this is a Quality of Implementation decision, not a Core C++ decision. Let the implementation decide if it can optimize the storage into read-only memory, and let users get their forward/move semantics for free.

Strongly in Favor.


### [p1255 - A view of 0 or 1 elements: `view::maybe`](https://wg21.link/p1255)

Steve Downey is part of _SG16: Unicode_ (as am I) and I didn't realize he was putting out this great paper! `view::maybe` gives us an adapter that answers a question I've seen posited by many individuals from reddit to Slack to Twitter and elsewhere: why can't we use a for loop / iteration over a `std::optional` or similar? Rather than modify each and every class like this to have iterators, or add crazy `std::begin`/`std::end` declarations in all the places we could possibly want this functionality, this paper does the smart thing. It lifts this into `view::maybe`, turning it into a reusable abstraction that can be extended and worked with in typical range algorithms. This is also useful for containers that hold optionals or pointers.

This is more of the kind of utility and design we need. It addresses a feature that people from Standard Library Maintainers to regular programmers want and comes up an elegant solution. The solution is also focused, not riddled with holes, and thankfully does not need to be patched over with language features either! Clean, well-designed, and well thought out. The only thing I would want to see in the future of this paper is wording that is generalized around the concept of any type that fits the necessary subset of [`Cpp17NullablePointer`](http://eel.is/c++draft/nullablepointer.requirements).

Strongly in Favor.


### [p1181 - proposing `unless`](https://wg21.link/p1181)

This paper solves a fundamental problem that has plagued the Standard Library Maintainers for ages: somebody overloads an operator and does something crazy/whacky/stupid with it. Did you know you can overload `operator!`? I didn't, until I saw a [C++ Standard Library Issue](https://cplusplus.github.io/LWG/issue2114) filed for this exact kind of bizarre code, and how it becomes nearly impossible for a Standard Library maintainer to not only specify the library but implement such things.

This proposal helps ease some of those woes by creating a `!` that can't be overridden: in particular, `if not ( expression... ) { ... }`. A side effect of this is greatly improving the readability of code where performing the exact negation of each element of `expression` (since it can be quite a mix of compound operations) is a bit annoying to have to perform. The benefit of this is that `not` is very much just a mechanical replacement of `!`, so the syntax boils down to `if !( expression... )`. There are no new tokens to parse, and as far as I can read this does not clobber or confuse the grammar any. Note that the negation applies only after the full evaluation of the expression and the result is converted to a boolean, leaving any potential for overriding `!` out of the equation.

This also applies to `if constexpr`. Thankfully, [p1073 - immediate functions](wg21.link/p1073) is proposing a new keyword that is not `constexpr!`, so `if constexpr !` is actually just fine despite the hesitations in the paper! [p1073](wg21.link/p1073) ended up with `consteval`.

I don't discuss p1073, but both of these papers receive a Strongly in Favor from me.


### And that's it for now!

There will be more papers to review. San Diego is coming up quickly, and I've read a lot more than I have written about. Hopefully, this little series will keep everyone up to date on some of the more interesting corners of things that are heading towards Standardization! Make sure to ping the paper authors with your ideas, so they know about it and come prepared to San Diego.

Do you have a paper you'd like me to review? Head down below to find my e-mail and other ways to contact me, and send me some recommendations. Although, I warn that if it is about Modules or Coroutines my eyes might glaze over.

Ta-ta for now! ♥
