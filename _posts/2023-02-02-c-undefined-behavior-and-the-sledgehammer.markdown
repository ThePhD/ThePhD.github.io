---
layout: post
title: "Undefined behavior, and the Sledgehammer Principle"
permalink: /c-undefined-behavior-and-the-sledgehammer-guideline
feature-img: "/assets/img/2023/02/sledgehammer.jpg"
thumbnail: "/assets/img/2023/02/sledgehammer.jpg"
tags: [C, Standard, Undefined behavior, ⌨]
excerpt_separator: <!--more-->
---

Previously, an article made the rounds concerning Undefined behavior that made the usual Rust crowd go nuts and the usual C and C++ people get grumpy that someone Did Not Understand the Finer Points and Nuance of Their Brilliant Language. So, as usual, it's time for me to do what I do best<!--more--> and add nothing of value to the conversation whatsoever.

It's time to get into The Big One in C and C++, and the Sledgehammer Principle.




# Undefined behavior

[This article](http://blog.pkh.me/p/37-gcc-undefined-behaviors-are-getting-wild.html) is the one that flew around late November 2022 with people having a minor conniption over the implications that GCC will take your grubby signed integer and exploit the Undefined behavior of the C Standard to load its shotgun and start blasting your code. The full code looks like this:

```cpp
#include <stdint.h>
#include <stdio.h>
#include <stdlib.h>

uint8_t tab[0x1ff + 1];

uint8_t f(int32_t x)
{
    if (x < 0)
        return 0;
    int32_t i = x * 0x1ff / 0xffff;
    if (i >= 0 && i < sizeof(tab)) {
        printf("tab[%d] looks safe because %d is between [0;%d[\n", i, i, (int)sizeof(tab));
        return tab[i];
    }

    return 0;
}

int main(int argc, char **argv)
{
    (void)argc;
    return f(atoi(argv[1]));
}
```

The "bad" code that GCC optimizes into a problem is contained in `f`, particularly the multiply-and-then-check:

```cpp
    // …
    int32_t i = x * 0x1ff / 0xffff;
    if (i >= 0 && i < sizeof(tab)) {
        printf("tab[%d] looks safe because %d is between [0;%d[\n", i, i, (int)sizeof(tab));
        return tab[i];
    }
    // …
```

This program, after being compiled VIA the use of GCC with i.e. `gcc -02 -Wall -o f_me_up_gnu_daddy`, can be run with as `./f_me_up_gnu_daddy 50000000` to which you will be greeted with a lovely segmentation fault (that will, of course, core dump, as is tradition). As the blog points out, it'll even `printf` out "a non-sense lie, go straight into dereferencing `tab`, and die miserably".

If you're following along, 50,000,000 multiplied by `0x1ff` (511 in decimal) results in 25,550,000,000; in other words, a number FAR too big for a 32-bit integer (whose maximum is meager 2,147,483,647). This triggers a signed integer overflow. But the optimizer assumes that signed integer overflow can't happen since the number is *already* positive (that's what the `x < 0` check guarantees, plus the constant multiplication). So, eventually, GCC takes this code and punches it in the face during its optimization step, and effectively removes the `i >= 0` check and all it implies. The blog post author is, of course, not happy about this. And the fact is,

they're not alone.




# The Great Struggle

First thing I'd like to point out is that this isn't the first time the C Language, implementations, or the C Standard came under flak for this kind of optimization. Earlier last year, someone posted the *exact* same style of code -- using a signed integer index and then trying to bolt safety checks onto it after doing the arithmetic -- and (at the time, before the Twitter Collapse and they locked their account) tagged it with the hashtag `#vulnerability` and said GCC was making their code more dangerous. Prior to that, Victor Yodaiken [went on an The-Committee-and-implementers-have-lots-their-marbles bender](https://www.yodaiken.com/2021/05/19/undefined-behavior-in-c-is-a-reading-error/) for about a year and a half, which culminated in his paper for how [ISO C was Not Suitable For Operating System Development](https://arxiv.org/pdf/2201.07845.pdf) (and even [published a video explaining his position](https://dl.acm.org/doi/10.1145/3477113.3487274) in Proceedings of the 11th Workshop on Programming Languages and Operating Systems).

And that's just the recent examples, because the same issues [go back a long way](https://gcc.gnu.org/bugzilla/show_bug.cgi?id=30475).

Given the number of times people have taken serious affront to compilers optimizing on Undefined behavior, you'd think WG14 — the C Committee — or WG21 — the C++ Committee — would make it their business to bring a solution to what has been a recurring issue in the C and C++ communities for decades now. But before we get into things that were done and should be done, we should talk about why everyone is increasingly freaking out about Undefined behavior, and why in particular it's starting to become a more frequently occurrence that has fellow system's programmers and compiler implementers getting upset and getting into staring contests to see who flinches first. The eyes get dry, the cycles start settling in, and it becomes difficult to keep our hands on the keyboard and focus on what's going on…

![An anthropomorphic sheep sits at a computer, eyes bleary and tired as they stare directly at the viewer in a dimly lit room. Their hand-hooves are on the keyboard, and their head is tilted to the side in tiredness, but they're trying to maintain and upright posture and keep staring as best as they can. A portrait in the room of an anthropomorphic with similar eyes stares at the viewer as well, somewhat creepily.](/assets/img/stratica/deeply-tired.png)

And, well. Unfortunately,



## We Blinked First

As Victor Yodaiken tries to point out in his blog post and presentation, he believes that Undefined behavior was not meant to be the tool that people (particularly, compiler implementers) are using it for today. The blog post linked above also is shocked, and cites the [Principle of least astonishment](https://en.wikipedia.org/wiki/Principle_of_least_astonishment) as reasoning to why GCC, Clang, and other compilers are being meanie-meanie-buttfaces for optimizing the code in this manner. And, the best possible reaction (in less an amusing sense and more of a "people really were depending on this stuff, huh?" sense), is from felix-gcc in a much older GCC bug report:

> > signed type overflow is undefined by the C standard, use unsigned int for the
> addition or use `-fwrapv`.
>
> You have GOT to be kidding?
> 
> …
>
> PLEASE REVERT THIS CHANGE.  This will create MAJOR SECURITY ISSUES in ALL MANNER OF CODE.  I don't care if your language lawyers tell you gcc is right.  THIS WILL CAUSE PEOPLE TO GET HACKED.
>
> — [felix-gcc, January 15, 2007](https://gcc.gnu.org/bugzilla/show_bug.cgi?id=30475#c4)

Users blinked in the staring contest, and in that brief moment GCC took all the liberty it wanted to begin optimizing on Undefined behavior. Clang followed suit, and now life is getting dicey for a lot of developers who believe they are writing robust, safe code but actually aren't anymore because they were using constructs that the C Standard defined as Undefined behavior. The language lawyers and compiler politicians had, seemingly, won out and people like Victor Yodaiken, felix-gcc, and bug/ubitux (the author of the blog post that sparked the most recent outbreak of protest against optimization) were left with nothing.

... Which is, of course, not the whole truth.

The truth is that, even as much as [Yodaiken argues in his blog that Undefined behavior being used for optimization is simply a "reading error"](https://www.yodaiken.com/2021/05/19/undefined-behavior-in-c-is-a-reading-error/), the problem did not start with a reading error. The problem started much earlier, when ISO C set you up, some of you before you were even born or knew what a computer was.




# Hands-off

WG14 had a problem.

There were a bunch of computers, and they had deeply divergent behavior. On top of that, a lot of things were hard to check or accommodate with the compute power available at the time. At that moment, enumerating all of the possible behaviors seemed to be a daunting task, bordering on impossible (and seriously, if people don't want to write documentation now it is hard to imagine how much people wanted to deal with it in the era of punch card mainframes). They also did not want to prevent different kinds of hardware to use their shiny new language, or the various uses different [kinds of computers would end up having](https://twitter.com/thingskatedid/status/1605969825964048389). So they devised a scheme, back in the day when there was effectively one compiler vendor per deeply disturbing/cursed architecture being built:

what if they just **didn't**?

Enter Undefined behavior. It was the Committee's little out; things that were too hard or they weren't sure about or it was just too damn hard to document, they deemed it Undefined behavior. Using 1's complement versus 2's complement? Undefined behavior at the tips of integer ranges. Using a different kind of shifter for your 16-bit integers and you shift the top bits? Undefined behavior. Pass a too-big argument into a function call meant to negate things? Undefined behavior! Multiply two integers together and they're not within the range anymore? That's right,

[Undefined behavior](https://www.tiktok.com/embed/v2/6912855387788102918).



## The Curse

WG14 got to wash their hands of the problem. And for the next 30/40 years, it stayed that way. Users, of course, couldn't just write programs on top of "Undefined behavior". So folks like felix-gcc, Victor Yodaiken, and perhaps hundreds of thousands of others struck what was effectively a backroom deal with their compiler implementers. Compilers would just "generate the code", and let users basically do "whatever they told the machine to do". This is the interpretation that Yodaiken ultimately tries to get us back to by performing the most tortured and drawn out sentence reading of all time in the above-linked blog post about Undefined behavior being a "reading error" in C. Whether or not anyone gets on — or wants to get on — the same grammatical train as he does, doesn't really matter because in reality there's a de-facto pecking order for how C code is interpreted. This ordering determines everything, from how Undefined behavior gets handled, to which optimizations trigger and which ones don't, to how Implementation-defined Behavior and Unspecified Behavior gets written down; EVERYTHING that you touch in your implementation The ordering – from most powerful to least powerful — when it comes to interpreting the behavior is as follows:

3. The Sum of Human Knowledge about code generation / interpretation
2. The Compiler Vendor / Implementer
1. The C Standard and other related standards (e.g., POSIX, MISRA, ISO-26262, ISO-12207)
0. The User (⬅ We Are Here)

As much as I would not like this to be the case, users – me, you and every other person not hashing out the bits 'n' bytes of your Frequently Used Compiler — get exactly one label in this situation.




# Bottom Bitch

That's right. When used in a conscientious and consenting relationship it's a lot of fun to throw this word around, but in the context of dealing with vendors, it really doesn't help when all they have to do is throw up the Stone Wall and say "sorry, Standard Said So" and make off like bandits. "But wait," you say, desperately trying to fight the label we've all been slapped with. "What about `-fwrapv` or `-ftrapv` or `-fno-delete-null-pointer-checks`? That's me, the user, being in control!" Unfortunately, that's not real control. That's something your *implementation* gives you; it's still entirely in their control, and frequently when you migrate to compilers outside of the niceties that GCC or Clang offer you, you can get shafted in exactly the same way. Take, for example, SDCC, which produces a different [structure size for this bitfield sequence in this bug report](https://sourceforge.net/p/sdcc/bugs/3530/). To be **very** clear here, SDCC is not wrong here; the C Standard allows them to do exactly this, and likely for the machines and compatibility they compile to this is exactly what they need to be doing. And, well, that's the thing.



## It's the Standard's Fault

Compiler vendors and implementers are always allowed to do whatever they want, and many times they break with standards to go and do their own things for compatibility reasons. But even though "the standard" is ranked beneath a compiler vendor and implementer, it's still a mighty sword to swing. While you, the user, are powerless in the face of a vendor, the Standard is an effective weapon you can wield to get the behaviors you want to do. This is not only where felix-gcc and ubitux failed, but where 30 years of C programmer communities failed. They lean too heavily on their implementers and these backroom, invisible deals, praying to some callous and capricious deity that their assumptions are not violated. But implementers have their own priorities, their own benchmarks, and their own milestones to hit. Every day of accepting whatever schlop an implementation handed us — whether it was a high quality Clang-level operational control through `#pragma`s and other schlop, or Compiler-written-by-a-drunk-EE-in-a-weekend schlop — was a day we condemned our own future.

For all the talk C programmers love to make about how "close to the metal" they are, they never really were. It was always based on that invisible contract that your implementer would "do the right thing", which as it turns out tends to mean different things to different people and different communities. While signed integer overflow optimizations based on UB make Benchmark Line Go Up, it has a noticeable impact on the ability to predictably handle overflow at a hardware level because the compiler vendor precludes you from trying in the first place by optimizing based on that.

This is why C and C++ programmers get so pissed off at GCC, or Clang, or whatever implementation doesn't do what they want the compiler to do. It shatters the illusion that they were in the driver's seat for their code, and absolutely violates the Principle of Least Astonishment. Not because the concept of Undefined behavior hasn't been explained to death or that they don't understand it, but because it questions the very nature of that long-held "C is just a macro assembler" perspective. And as we keep saying, year over year, GCC bug after GCC bug, highly-upvoted blogpost after highly-commented writeup, that perspective isn't going to change because it's a fundamentally-held belief of the C and — to a lesser extent — the C++ community. "Native" code, "Machine" code, inline "assembly", "close to the metal"; all of it is part of that shiny veneer of being the most badass person with a computer and access to a terminal in any given room.

And compiler vendors doing this threatens the programmer's sacred and holy commune that is meant to be between them and the hardware.




# Fighting Back

If you've been visiting this blog long enough, you know that we don't do JUST problems. We're engineers, here. Compiler vendors and implementers are not evil, but they've clearly drawn their line in the sand: they are going to optimize based on Undefined behavior. The more of it we have in the standard the more at-risk us we-tell-the-hardware-what-to-do folks are going to be. If compiler vendors are going to start flexing on us and optimizing UB, then there's a few things we can do to take back what belongs to us.



## Adopting the 'Sledgehammer Principle'

I've personally developed a very simple rule called the 'Sledgehammer Principle'. Whenever you commit undefined behavior, you have to imagine that you've taken a sledgehammer and smashed it into a large, very expensive vase your mother (or father, or grandmother, or whoever happens to be closest to you) owns. This means that if you needed that vase (e.g., the result of that signed integer multiply), you have to imagine that it is now completely unrecoverable. The vase is goddamn smashed, after all; only through brutally difficult craftsmanship could you ever imagine putting it back together. This means that, before you swing your sledgehammer around, you check **before** you commit the undefined behavior, not afterwards. For example, taking the program from the blog post, here's a modification that can be done to check if you have fucked everything up before things go straight to hell:


```cpp
#include <stdint.h>
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <limits.h>

uint8_t tab[0x1ff + 1];

uint8_t f(int32_t x)
{
    if (x < 0)
        return 0;
    // overflow check
    if ((INT32_MAX / 0x1ff) <= x) {
        printf("overflow prevented!\n");
        return 0;
    }
    // we have verified swinging this
    // particular sledge hammer is a-okay! 🎉
    int32_t i = x * 0x1ff / 0xffff;
    if (i < sizeof(tab)) {
        printf("tab[%d] looks safe because %d is between [0,%d) 🎊\n", i, i, (int)sizeof(tab));
        return tab[i];
    }
    else {
        printf("tab[%d] is NOT safe; not executing 😱!\n", i);
    }
    return 0;
}

int main(int argc, char* argv[])
{
    (void)argc;
    memset(tab, INT_MAX, sizeof(tab));
    return f(atoi(argv[1]));
}
```

This is [a safe way to check for overflow](https://godbolt.org/z/7hz95zncd), **before** you actually commit the sin. This particular overflow check doesn't have to worry about negative or other cases because we check for "less than zero" earlier, which makes this particularly helpful. The coding style of the above snippet is not great, of course: we're using magic numbers and not specifying things, but this gets across the general idea of the Sledgehammer Principle: check **before** you swing, not **after** the vase is broken. But that's only a minor balm on the overall issue, truly.



## Why is doing Integer Multiplies like swinging a Sledge Hammer?

This is, of course, the crux of it all. Why is doing something so simple — especially something that was perfectly within the purview of "I can check it post-facto in my hardware" — so difficult? And the answer of course lies above, in the fact that the compiler implementers are the Top Dogs in this scenario and we, the users, are still at the bottom of the power ranking. We're not Super Saiyan Goku ready to take on Super Broly in an epic battle; we're the weak, pathetic Krillin of the of the whole shebang [that's about to get punched/slapped/destroyed in for comedic effect](https://www.youtube.com/watch?v=5jGKkTcgbRE).

So, how do we get this sledgehammer out of our hands? How do we make it so every vase we touch does not have the potential to shatter into a million irreplaceable pieces?




## Our Greatest Weapon

Notice how every single bug report linked in this blog post ends with "the standard says we can, no I am not joking, take a hike" (not exactly with that *tone*, but you get the idea). If these vendors are going to be all about conforming to the C Standard, then what we need to start doing is investing in changing or adding to the Standard so we can start having it reflect the behaviors we want. It truly sucks that K&R left so much Undefined in the first place, and that the first iteration of the ANSI C Committee left it that way, and it snowballed into hundreds of places of Unspecified and Undefined behavior that are exploitable by not just the Committee, but by red teamers that know how to abuse the code we write daily.

This is not a hopeless situation, however. In C, we finally standard the `<stdckdint.h>` header thanks to David Svoboda's tireless efforts to produce safer, better integers in C. I wrote up about its usages [here](/c-the-improvements-june-september-virtual-c-meeting#n2683---towards-integer-safety), but it may take too long to get here. If you're not interested in waiting, you can grab a publicly-available version of the code written to a pretty high quality [here](https://gitlab.com/Kamcuk/ckd/) (C++-heads can grab Peter Sommerlad's [Simple Safe Integer library themselves](https://github.com/PeterSommerlad/PSsimplesafeint), since C++ itself hasn't made any progress in this area). It won't be perfect everywhere, but that's the fun of open source code; everyone can make it a little better, so we can all stop reimplementing foundational things five hundred times and start actually getting **really** high quality goods out of things. David managed to change the C Standard for the better, and while it won't fix everything it does set a precedent for how we can make this a tractable problem that is solvable before most of us retire. There is, of course, a lot more to do:

- `ckd_div` was not included in David's proposal. This is because the only case of failure for division is ` / 0` and `{}_MIN / -1`, because the result would not be representable in a 2's complement integer (`{}_MAX` of any given integer type does not fit there).
- `ckd_modulus` has the exactly same problems as `ckd_div`, and so anything that solves `ckd_div` can bring `ckd_modulus` along for the ride.
- `ckd_right_shift` and `ckd_left_shift` are both not included. There is Undefined behavior if we shift into the high bit; it would be very nice to provide definitions for these so that there is well-defined behavior on a shift that happens to move bits into the high bit for a signed integer, especially since we now have 2's complement behavior in C.

This, of course, only covers problematic mathematics for C and C++-style integers. There a TON of other Undefined behaviors that cause serious issues for users, least of all being the fact that things like `NULL + 0` are Undefined behavior or passing `NULL` with a length of `0` to library functions (representing e.g. an empty array) is also Undefined behavior.

Integer promotion also tends to be a source of bugs (e.g., right-shifting a 16-bit `unsigned short` with the value `0xFFFF` by `15` results in an integer promotion to `int` before shifting the high bit into the top bit, resulting in undefined behavior). This is solved by using Erich Keane, Aaron Ballman, Tommy Hoffner and Melanie Blower's recently-standardized `_BitInt(N)` type, which I [also wrote about here](/c-the-improvements-june-september-virtual-c-meeting#n2709---adding-a-fundamental-type-for-n-bit-integers). This is something C++ doesn't currently have except in the form of bespoke libraries; we'll see if they move in this space but for now using the C thing in C++ such as in e.g. Clang might be a good way to get the non-promoting integers you need to take back control of your code from problematic behavior. (A slight warning, however: we have not solved `_BitInt(N)` for generic functions, so it will not work `<stdckdint.h>`'s generic functions since we do not have generic paramtricity in C.)




# Do Not Go Quietly

It's a lot of work, and we have to keep actively working at it, but this is the world we inherited from our forebears. It's the one where we're the least of the ecosystem, and where vendors and implementers still have outsize control of what goes on. But if they are going to hold the C Standard up as the Holy Text that justifies everything they do, then if we want to survive we need to flip the script. A lot of people don't want fixes or changes or C. They like stability, they like things being frozen. They view it as a feature when it takes 5 years to put something as simple as `#embed` into C because that's how we stop C from becoming broken. But, for me and many others trying to write to the hardware, trying to write to the metal, trying to write code that says what we mean,

C and C++ are already broken.

We can either leave it like this and keep letting the vendors take our space from us, or we can fight back. I threw money into a bus ticket and slept in a rickety room where I got to see some very large spiders and a few other many-legged bugs say good morning to me on my curtain every morning [just for the chance to get into this mess of a language](/follow-the-river-wg14-ithaca-2019), because it doesn't have to stay broken. We deserve signed integers that don't trigger undefined behavior on a flippin' left shift. We deserve multiplies and subtracts and adds that don't punch us in the face. We deserve code that **actually** respects what we write and what we say when we say it, so we can build the necessary safety guarantees into what we ship to what I know C programmers are shipping to millions of people all over the globe.

We deserve better, and if our forefathers aren't going to settle the question and give it to us then we better damn well do it ourselves.




# Of Course …

There is a secret option as opposed to all of this. You could not remember the Sledgehammer Principle. You can ignore the minutia of Undefined or Implementation-defined or Unspecified or whatever behavior. You could simply just use tools

[that don't treat you like you're too stupid to write what you mean](https://godbolt.org/z/Gn75WMx44). 💚


Art by the ever-skillful [Stratica](https://stratica.carrd.co/)!
