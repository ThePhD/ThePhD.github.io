---
layout: post
title: Why the C Language Will Never Stop You from Making Mistakes
permalink: /your-c-compiler-and-standard-library-will-not-help-you
feature-img: "assets/img/2020-08-09/pexels-energepiccom.jpg"
thumbnail: "assets/img/2020-08-09/pexels-artyom-kulakov.jpg"
tags: [C, 📜]
excerpt_separator: <!--more-->
---

Short answer: because we said so.<!--more-->

:)

# ... What?

Alright, fine, that's too short to make it an article, dear reader, and my inflammatory words demand an explanation.

The C Committee meeting -- originally scheduled for Freiburg, Germany but not happening there because _(* vague gesturing towards outside *)_ -- concluded Friday, August 7th. It went pretty well, with us making good progress on all fronts. Yes, we make real progress; I promise we do, and this is not a dead language.

As a side note I've also become the Project Editor for C so before you take this as an uninformed rant of a person too lazy to "make things better", let me assure that I am, indeed, incredibly invested in making sure C can begin to meet the needs of developers without needing 50 vendor-specific extensions to build remotely nice/usable libraries and applications.

Still, I made a claim (the C Language will never stop you from making mistakes), and therefore I need to substantiate it. We could look at the thousands of CVEs and the inherent issues with tons of C Code, or the need for MISRA to vigorously police every last little potential C feature to prevent misuse ([hello, K&R prototype declarations](https://twitter.com/thephantomderp/status/1290824683399663616)...), or other more intricate and fun bugs related to portability and undefined behavior. But instead, we are just going to take the words from the horse's mouth: the Committee itself.


# Ooh, 🍿 time?!

No, dear reader, put the popcorn away. As with all ISO proceedings, I am not allowed to quote anyone verbatim, and this is not a name-and-shame article. It will, however, explain why things that we can easily diagnose and identify as bad behavior in Standards-conforming ISO C will never go away. And we will start with a paper by Dr. Philipp Klaus Krause:

[N2526, use const for data from the library that shall not be modified](http://www.open-std.org/jtc1/sc22/wg14/www/docs/n2526.htm).

N2526 is a very simple paper. "Some data returned by the library is `const` morally, spiritually, and in fact even by its implementation. It's undefined behavior and wrong to write into it, so let's stop teasing each other about it and put a ring on that finger bad boy!" ... Okay, that's not exactly what it says, but I'm sure the idea makes sense to you, dear reader. Originally, when the vote for this paper was taken, there was almost no votes against the paper. Then, a few people voiced their _strong_ objections to the paper because it breaks old code. And, well, of course that's bad: even my breath caught and I strained to think about this -- adding `const`? C has no ABI that could be affected by this, C doesn't even respect qualifiers, how are we breaking things?! So, let's talk about why this would be a breaking change in the eyes of certain folk.



# The C Language

Or, as I like to call it, "The Type Safety Is For Losers Language". Of course, that's a mouthful, so "C" will have to do. You may be wondering why I would call a language like C as having no type safety. After all,

```c
struct Meow {
	int a;
};

struct Bark {
	double b;
	void* c;
};

int main (int argc, char* argv[]) {
	(void)argc;
	(void)argv;

	struct Meow cat;
	struct Bark dog = cat;
	// error: initializing 'struct Bark' with an expression of incompatible type 'struct Meow'

	return 0;
}
```

I mean, let's be honest: that looks like strong type safety to me, Jim! Of course, it only gets spicier from here:


```c
#include <stdlib.h>

struct Meow {
	int a;
};

struct Bark {
	double b;
	void* c;
};

int main (int argc, char* argv[]) {
	(void)argc;
	(void)argv;

	struct Meow* p_cat = (struct Meow*)malloc(sizeof(struct Meow));
	struct Bark* p_dog = p_cat;
	// :3
	
	return 0;
}
```

f̵̞͟͞o̷r̰͇͎̞ ̡̝͍͍̥̗̖͖̲̮i͕͔̭̹̪̳͙͍̩ͅ ̷͖̙͈̗̯̙͓̲̻̩̱͖̺a̧̪̳̠̰̱͎̖̙̳̱͎̯̦̟͟m̶͓͔͉̺̩̙̖̞̼͈̖͇͇͇ ͇̘̬͕̯̕͢d͚o̢̳̱̙͠ͅg̴̨̦̬̥͓̯̩͞,̷̬͕͖͎̺͈̰̳̮͓̞ ͚d͎̀̕͢es̵̱͙̪͓͖͇̩̳̻͉͉̹̣͈͜͝t͖͈͉̩̭̙̺̬r̤̟̫͔̩̰̗̤͔̭̰̞̦̺ͅo̝̹̻ͅy̸͎͎͚̖̤̣̥̭̼̳͎e̘̟̰̣̫̹͇͍̦͓ͅr̙̙̳̠͔̙̖̯̺ͅ ̸̜̫̫̩̫̝͎̩̠̦̭̠̟o҉̛͢f̫̝͇ ҉̣̞̟a̪̲͎͇̣̠͖͖̮̥ͅl̷̛͈̦̰̜͍̲̟̬̘̝͕̰̮͠l̮̯̘͉̦͔̘̲̰̤͔̬̲͔

<br/><br/>
Yes, two entirely unrelated pointer types can be set to one another in standards conforming C. Most compilers warn, but this is standards-conforming ISO C code that is required to not be rejected unless you crank up the `-Werror` `-Wall` `-Wpedantic` etc. etc. etc.

In fact, other pointer-related things the compiler can accept without an explicit cast are:

- `volatile` (who needs those semantics anyways?!)
- `const` (write into all the read-only data you like!)
- `_Atomic` (thread safety, shred shmafety!)

Now, note I am not arguing that you should not be able to do these things at all. But when you're working in C -- wherein it is very easy to get yourself a 500, 1000-line function alongside variable names that do not describe WTF they are for -- the Hard Facts™ is that you work primarily with pointers, and you lack any amount of safety at all insofar as the Core Language cares. (NOTE: It **is** a constraint violation, but there's so much legacy code that every implementation ignores qualifiers anyhow, so your code won't ever _stop_ compiling because of this ([thanks, @fanf!](https://twitter.com/fanf/status/1292826424345337868))!) Every potential failure here is diagnosable -- easily so -- with a compiler, and they all warn, but they will never require you to cast to communicate to the compiler that you Seriously, Really Meant It. It also means -- much more importantly -- that the human beings who come after you will have no bloody idea if you meant to do this, either.

All someone has to do is strip the build of `-Werror` `-Wall` `-Wpedantic` and you'll be off to the races in committing multithreading, read-only, and hardware register crimes.

Now, this is fair, right? If someone strips off all of these warning/error flags, they obviously do not care about whatever faux pas or silly gaffe you've committed. Which means that, at the end of the day, these warnings are ultimately irrelevant and harmless as far as Standards-conforming, ISO C is concerned. And yet...



# We Consider Warning Changes, Breaking

Yes.

This is a special kind of hell that C developers have grown comfortable with, and to a lesser extent C++ developers. Warnings are seen as annoyances; and, as is shown by ever turning on `-Weverything` or `/W4`, they very well are. Shadowing warnings from variables in the global namespace (thanks, every header and C library ever is now a problem), using "reserved" names (as the kids say, "[lol nice one xd!!](https://wg14.link/n2493)"), and "this struct has padding because you used `alignof`" (... yes, yes I know it has padding, I explicitly asked it to increase my padding BECAUSE I USED `alignof` MR. COMPILER) are all incredibly time-wasting.

But they are warnings.

Even if they are annoying, they prevent problems. The fact that I can blithely [ignore all qualifiers and dumpster all forms of read, write, thread, and read-only safety](https://godbolt.org/z/rEbPvd) is a serious issue when it comes to communicating intent and preventing bugs. Even the old K&R syntax produced bugs in industrial and government codebases because users did the wrong thing. This is not because they are shit programmers: it's because they are straddling codebases sometimes older than they are and gearing up to ride on a multi-million line technical liability. You cannot keep the entirety of the codebase in your head: this is what conventions, static analysis, high warning levels, and everything are supposed to combat. Unfortunately,

everyone likes to have Warning Free code.

This means that the moment a GCC developer makes a warning more sensitive to potential problematic cases, maintainers (not the original developers) will suddenly get hefty several-gigabyte logs containing all the newest vomit of warnings and other things from their old code. "This is stupid", they say, "the code has been running for Y E A R S, why is GCC complaining now?". This means that even doing something like adding `const` to a function signature, even if it is morally, spiritually, and factually the right thing to do, is avoided. "Breaking" people is "they now have to look at code which has questionable intentions". This is code that would -- on pain of Undefined Behavior -- crash your chip or [corrupt your memory](https://twitter.com/MalwareMinigun/status/1260061878614556684). But that's the other problem with being a C developer in today's ecosystem, really.


### Age as a Measure of Quality

How many people would have ever guessed `sudo` had a vulnerability as jaw-droppingly simple as "-1 or integer overflow gives you access to everything"? How many people thought Heartbleed would be a real problem? How many game developers are shipping stb's "tiny" libraries without ever having run a fuzzer on them and realizing they contain more significant exploitable input vulnerabilities than they could imagine? This is not a dig at any of these pieces of code or the programmers behind them: they are providing a vital service that the world has depended on for decades, often with little to no support from anyone until a big problem exploded. But, the people worshipping and deploying these pieces of code end up fermenting a toxic adage under the auspice of survivorship bias:

"This code is so old and used by so many, how could it possibly have problems?"

By holding backwards compatibility and "not being a bother" as the highest ideals for C code, people who survive long enough in the industry begin to equate age with quality, as if codebases were barrels of wine in a cellar. The older and longer the code is used, the finer and more delectable the wine.

The reality is, unfortunately, far less romantic and cute: crawling with more bugs than you can shake a stick at, security vulnerabilities by the basketful, these technical liabilities only grow more risky with each passing day. Each system evolves into a half-alive, unaddressed, and partially-unmaintained rotting husk. They are gussied up and given the veneer of nobility and greatness when it is in fact an embalmed corpse, just waiting to be poked the wrong way before its festering, antediluvian pustules burst and fill your application with its finely aged botulism.


### Okay... Gross, but like what about the C Standard?

The problem I have noticed in my (incredibly short) tenure as a member is that we prefer backwards compatibility over all else, attributing old applications and their use cases over any chance at improving the safety, security, or craftsmanship of the C code for those who are moving to C, even today. Dr. Krause's paper is so small as to be almost inconsequentially uncontroversial: if someone does not like the warnings, they can shut them off. They are warnings, not errors, for a reason: no diagnostic is _required_ by the C abstract machine, ISO C allows that this code is accepted under the strictest build modes, and it would help the rest of the world to wean themselves off of APIs that clearly state "modifying the contents of what we hand back to you is undefined behavior".

And yet, we turned our opinion for the paper over anyways after "we cannot introduce new warnings" was brought up as a reason.

The argument against was essentially "there is so much code that will break if we change these signatures". This, again, frames a change in warnings as a break in behavior (remember, implicit conversions that remove qualifiers -- even `_Atomic` -- are perfectly valid ISO C even if it is a constraint violation). Were this the case, every compiler developer would have to introduce something similar to Rust's Epochs, But Just For Warnings to give people a "stable" set to always test against. This is not exactly a new sentiment: I have read papers from some of Coverity's own engineers about their handling of producing new warnings and the customer response to them. Managing "developer confidence" in new warnings and other things is tough work. It takes a long time to bootstrap developers on its usefulness. Even John Carmack took some time to get a proper set of warnings and errors suitable for his development out of his static analysis tools, [before concluding that "**it is irresponsible to not use it**"](https://www.gamasutra.com/view/news/128836/InDepth_Static_Code_Analysis.php).

And yet, as a Committee, we struggle with adding `const` to 4 function return values because it would add warnings to potentially dangerous code. We raised objections to deprecating the old K&R syntax despite tangible evidence of both innocent screw ups and also tangible vulnerabilities from developers passing in the wrong types. We _almost_ added undefined behavior to the preprocessor, just to bend over backwards and make a bespoke C implementation "do the right thing". We are always teetering on the edge of doing the objectively wrong thing for backwards compatibility reasons. And this, dear reader, is what scares me the most about the future of C.



# The C Standard Does Not Protect You

Make no mistake: no matter what programmers tell you or what people whisper in your ear, the behavior of C's governing body is very clear. We will not introduce warnings into your old code, even if that old code could be doing something dangerous. We will not steer you away from mistakes, because that could shake the veneer that what your old code does is, in fact, wrong. We will not make it easier for new programmers to write better C code. We will not demand that your old code is held to any Standard. Every new feature we add we will make optional, because we cannot possibly imagine holding compiler writers to a higher standard nor expect more out of our Standard Library vendors.

We will let the compiler lie to you. We will lie to your code. And when things do go bad -- a mistake, an "oopsie kapoopsie", a leak of data -- we will shake our heads solemnly. We will offer our thoughts and prayers and say "well, that's a shame". A shame, indeed...

Maybe we'll fix it another day, dear reader. 💚
