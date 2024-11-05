---
layout: post
title: "5 Years Later: The First Win"
permalink: /5-years-later-the-first-big-unicode-win-omg-yay
feature-img: "/assets/img/2024/10/champ-banner.jpg"
thumbnail: "/assets/img/2024/10/champ-banner.jpg"
tags: [C, C standard, 📜, Unicode, Success, OMG]
excerpt_separator: <!--more-->
---

[N3366 - Restartable Functions for Efficient Character Conversions](/_vendor/future_cxx/papers/C%20-%20Restartable%20and%20Non-Restartable%20Character%20Functions%20for%20Efficient%20Conversions.html) has made it into the C2Y Standard (A.K.A., "the next C standard after C23"). And one of my longest struggles — the sole reason I actually came down to the C Standards Committee in the first place —<!--more-->has come to a close.




# Yes.

When I originally set out on this journey, it was over 6 years ago in the C++ Unicode Study Group, SG16. I had written a text renderer in C#, and then in C++. As I attempted to make that text renderer cross-platform in the years leading up to finally joining Study Group 16, and kept running into the [disgustingly awful APIs for doing text conversions in C and C++](/cuneicode-and-the-future-of-text-in-c#part-of-this-post-will-add-to-the-table-from-part-1-talking-abou). Why was getting e.g. Windows Command Line Arguments into UTF-8 so difficult in standard C and C++? Why was using the C standard functions on a default-rolled [Ubuntu LTS at the time](/cuneicode-and-the-future-of-text-in-c#the-first-and-most-glaring-problem-is-what-happens-if-the-execut) handing me data that was stripping off accent marks? It was terrible. It was annoying. It didn't make sense.

It needed to stop.

Originally, I went to C++. But the more I talked and worked in the C++ Committee, the more I learned that they weren't exactly as powerful or as separate from C as they kept claiming. This was especially when it came to the C standard library, where important questions about `wchar_t`, the execution encoding, and the wide execution encoding were constantly punted to the C standard library rather than changed or mandated in C++ to be better. Every time I wanted to pitch the idea of just mandating a UTF-8 execution encoding by default, or a UTF-8 literal encoding by default, I just kept getting the same qualms: "C owns the execution encoding" and "C owns the wide encoding" and "C hasn't really separated `wchar_t` from its historical mistakes". And on and on and on. So, well.

I went down there.

Of course, there were even more problems. Originally, [I had proposed interfaces](https://www.open-std.org/jtc1/sc22/wg14/www/docs/n2431.pdf) that looked fairly identical to the existing set of functions already inside of `<wchar.h>` and `<uchar.h>`. This was, unfortunately, a big problem: the existing design, as enumerated in presentation after presentation and blog post after blog post, was truly abysmal. These 1980s/1990s functions are wholly incapable of handling the encodings that were present even at 1980, and due to certain requirements on types such as `wchar_t` we ended up creating problematic functions with unbreakable [Application Binary Interfaces (ABIs)](/to-save-c-we-must-save-abi-fixing-c-function-abi).

During a conversation on very-very-old Twitter now, I was expressing my frustration about these functions and how they're fundamentally broken. But that if I wanted to see success, there was probably no other way to get the job done. After all, what is the most conservative and new-stuff-hostile language if not C, the language that's barely responded to everything from world-shattering security concerns to unearthed poor design decisions for some 40 years at that point? And yet, [Henri Sivonen](https://hsivonen.fi/) pointed out that going that route was still just as bad: why would I standardize something I know is busted beyond all hope?

Contending with that was difficult. Why should I be made to toil due to C's goofed up 1989 deficiencies? But, at the same time, how could I be responsible for continuing that failure into the future in-perpetuity? Neither of these questions was more daunting than the fact that what was supposed to be a "quick detour" into C would instantly crumble away if I accepted this burden. Doing things the right way meant I was signing up for not just a quick, clean, 1-year-max brisk journey, but a **deep** dungeon dive that could take an unknown and untold amount of time. I had to take a completely different approach from `iconv` and `WideCharToMultiByte` and `uconvConvert` and `mbrtowc`; I would need to turn a bunch of things upside down and inside out and come up with something entirely new that could handle everything I was talking about. I had to take the repulsive force of the oldest C APIs, and grasp the attractive forces of all of the existing transcoding APIs,

<center>and unite them into something entirely different and powerful…</center>

<br/><br/>

<center><img src="/assets/img/zerospanda/Purple.png" alt="An anthropomorphic sheep wearing a purple robe with a blue scarf stares intently and directly at the viewer, pupils solid and without light with the whites of their eyes fully showing. Their hand it extended towards the viewer, with their thumb and pinky extended out while their ring and middle fingers and curled in. The index finger is curled in, but less so and rests on top of the ring and middle finger, triggering the ancient Imaginary Technique. Bright light emits from the meeting point of the index, ring, and middle fingers just above the palm, ready to unleash the Great Energy."/></center>

<center><sub>Imaginary Technique: Cuneicode</sub></center>




# Henri was right.

It took a lot of me to make this happen. [But, I made it happen.](/cuneicode-and-the-future-of-text-in-c#static-conversion-functions-for-c) Obviously, it will take some time for me to make the patches to implement this for glibc, then musl-libc. I don't quite remember if the Bionic team handling Android's standard library takes submissions, and who knows if Apple's C APIs are something I can contribute usefully to. Microsoft's C standard library, unlike its C++ one, is also still proprietary and hidden. Microsoft still does a weird thing where, on some occasions, it completely ignores its own Code Page setting and just decides to use UTF-8 only, but only for [very specific functions](https://learn.microsoft.com/en-us/cpp/c-runtime-library/reference/mbrtoc16-mbrtoc323) and [not all of them](https://learn.microsoft.com/en-us/cpp/c-runtime-library/reference/mbrtowc).

I GENUINELY hope Microsoft doesn't make the mistake in these new functions to not provide proper conversions to UTF-8, UTF-16, and UTF-32 through their locale-based execution encoding. These APIs are supposed to give them all the room to do proper translation of locale-based execution encoding data to the UTFs, so that customers can rely on the standard to properly port older and current application data out into Unicode. They can use the dedicated UTF-8-to-UTF-16 and vice versa functions if needed. The specification also makes it so they don't have to accumulate data in the `mbstate_t` except for radical stateful encodings, meaning there's no ABI concerns for their existing stuff so long as they're careful!

But Microsoft isn't exactly required to listen to me, personally, and the implementation-defined nature of execution encoding gives them broad latitude to do whatever the hell they want. This includes ignoring their own OEM/Active CodePage settings and just forcing the execution encoding for specific functions to be "UTF-8 only", while keeping it not-UTF-8 for other functions where it does obey the OEM/Active CodePage.




# All in All, Though?

The job is done. The next target is for [P1629](https://wg21.link/p1629) to be updated and to start attending SG16 and C++ again (Hi, Tom!). There's an open question if I should just abandon WG14 now that the work is done, and it is kind of tempting, but for now... I'm just going to try to get some sleep in, happy in the thought that it finally happened.

We did it, chat.

A double-thanks to TomTom and Peter Bindels, as well as the Netherlands National Body, NEN. They allowed me to attend C meetings as a Netherlands expert for 5 years now, ensuring this result could happen. A huge thanks to all the [Sponsors](https://github.com/users/ThePhD/sponsorship) and [Patrons](https://www.patreon.com/Soasis) too. We haven't written much in either of those places so it might feel barren and empty but I promise you every pence going into those is quite literally keeping me and the people helping going.

And, most importantly, an extremely super duper megathanks [h-vetinari](https://github.com/h-vetinari), who spent quite literally more than a year directly commenting on every update to the C papers directly in my repository and keeping me motivated and in the game. It cannot be understated how much those messages and that review aided me in moving forward.

God Bless You. 💚

- Banner and Title Photo by [Coco Championship, from Pexels](https://www.pexels.com/photo/boxing-winner-inside-boxing-ring-598687/)
- Imaginary Technique: Purple Image by [ZerosPanda (NSFW Artist, Careful Clicking Through!)](https://twitter.com/PandaZeros/status/1735018900822049206)

{% include anchors.html %}
