---
layout: post
title: The San Diego 2018 Aftermath
permalink: /san-diego-2018-c++-committee-trip-report
feature-img: "assets/img/2018-12-03/beach-dawn.jpg"
thumbnail: "assets/img/2018-12-03/beach-dawn.jpg"
tags: [C++, Unicode, text processing, future, sg-16, embed, out_ptr, standards, proposals, 🤝, ⌨️, 📜]
excerpt_separator: <!--more-->
---

I'm not going to give you an overview of everything that happened at the meeting. I had some 8+ papers and I needed to focus on my own plus shepherd a few ones I volunteered myself for. [Other trip reports](https://www.reddit.com/r/cpp/comments/9vwvbz/2018_san_diego_iso_c_committee_trip_report_ranges/) go into depth about the overall events: I'm going to focus on what I saw, what I worked for, what I pushed through, and what I learned!

Let's dig in.<!--more-->

I've talked about the Committee before and its processes. This blog post is going to contain some of the information about the inner workings of the Committee, but it will mostly talk about proposals, proposals, proposals. Some of the more fun things I did will be interspersed around other topics!

I'm going to use some emoji to depict the state of all the proposals I talk about or are interested in. It'll work out something like this:


🛡️ - Successfully championed / defended.

⚔️ - Not dead, but under contention.

🚑 - Not killed/rejected, but needs (emergency) attention!

🗡️/☠️ - Leave it alone it's already dead...!


# Things Unseen

My deepest apologies to Rein Halbersma and Cicada for not getting [p0330 - Literal Suffixes for size_t and ptrdiff_t](/vendor/future_cxx/papers/d0330.html) looked at in time; I could not be in EWGI while the `std::out_ptr` review in LEWGI was ongoing to take advantage of overflow time. p0330 remained unscheduled throughout the week. The good news is I will still advocate for Literal Suffixes in Kona 2019, since it is an absolutely tiny and reasonable feature that probably shouldn't be shuttered into C++23 since it was originally almost put into C++17 before the original author ran out of steam. I also sent an e-mail out to the internal reflector to get some bikeshed discussions out of the way as well. Still, we are in a feature freeze so it may just be objected or not seen on procedural grounds, and it'll end up in C++23 anyways.

On the bright side, I have [some cool, hip slides](/vendor/future_cxx/papers/presentations/d0330.pdf) for the feature already so at least my work is done in that department.

# EWGI Battle Standings

I saw only EWGI because none of my papers made it out and were viewed by EWG in time. This is a bit of a disappointment, but there is still room to get things seen in Kona. 

### [std::forward from std::initializer_list - p1249r0](https://wg21.link/p1249r0)

☠️ - **Severely wounded. Cause: The Three Letter Demon.**

I presented this paper on behalf of Alex Christensen, the proposer of p1249r0. And I did a damn good job presenting it. I had the room eating out of the palm of my hands, heads were nodding, people were saying they liked the paper and presentation. Oh man, my first EWG(I) paper and the room was loving it. A hand was raised. I was all smiles. Success, so clearly in view! Or, was it merely —

"This breaks ABI."

— a trick of the light...?

Indeed, the success was nothing more than an illusion just waiting for my first encounter with the hideous Leviathan that is ABI. Despair dropped into the the pit of my stomach. My hands grew clammy. My mind raced wildly and crazily. All of my steam, all [of my presentation's](/vendor/future_cxx/papers/presentations/d1249.pdf), down the drain. C++ doesn't define an ABI, but every vendor has one and right now what we do is simple: promise infinity binary compatibility forever and ever, never breaking users with backwards compatibility stories for millennia. This means that if you compile C++98 code with a `std::string` it doesn't matter if you learned things in the last 20 years: you can build and link an application today with `std::string` in the interface and it'll work. 

ABI sucks. But it's what vendors abide by and what keeps companies wanting to have long-term investment in C++. I hope that every language that comes after C++ focuses on how to make clean ABI changes or provide definite ABI-stable things that people can use in their binary interfaces. Hell, making it so there are things marked for ABI stability that warn you when they're used in exported DLLs and stuff would be a pretty great first step.

What makes this even more sad is that this ABI break is due to a premature optimization created when the Committee originally shipped a `std::initializer_list` and its Core Wording.

By making it backed by an "array of **const** `E`", compilers were free to assume that nobody would move out of the storage or touch the storage in any meaningful fashion. This means they could essentially perform only a single allocation of an `initalizer_list`'s backing array storage for a loop, rather than create a new `initializer_list` with every loop (so long as the `initializer_list`'s contents did not depend on something in the loop's iteration or function parameters or runtime local variables, etc. etc. the list of cases goes on).

If we were to make p1249's in C++20 or C++23, the destruction would be subtle and full of the best Stack Overflow questions filled to the brim with confusion: someone taking a `std::initializer_list` into a DLL compiled with C++20 or later with p1249r0 would move out of your C++17 or below application's `std::initializer_list`. The storage would not be renewed since the compiler will assume it will never be moved out of, only copied out of. And thusly, everything goes to hell in your main application while the C++20 DLL happily chugs along.

It's a convoluted scenario but chances are someone in the millions of C++ programmers would trip that mine, and they'd be upset. "C++ is terrible." "C++ betrayed me." "What is the Committee doing?" "What's wrong with GCC, why would they do this?" "How could Clang allow this change into the compiler, this is crazy!" And so on, and so forth.

I wish we would keep the API stable, but commit to breaking the ABI every 3 standards cycles. 9+ years is a lot of time to buy people so that we can ship better Boyer-Moore searchers, better iostream implementations, better `initializer_list` and better... well, better damn everything. Inline namespaces were supposed to make it easier to cause linker errors when you linked across incompatible boundaries because full mangled names would be different, but... well. Still nothing official. Plus, VC++ a bit ago moved sizable chunks of its Common Runtime -- which it used to break every major release -- into the OS. So now it's impossible to change those things until the OS is ready to take a break (i.e. once in a decade, maybe???)...

It's an interesting situation we have ourselves here, that's for sure.

The way forward for this paper is simple. Keep `std::initializer_list`. Make `std::init_list`. Duplicate every `std::initializer_list` constructor to one that takes a `std::init_list` for the entire Standard Library. Add an implicit conversion from `std::init_list` to `std::initializer_list` Then, apply the new rules to the core wording from p1249. Then we can finally have not-broken `init_list`.

I hope.


### [Pointer to Member Functions and Member Objects are just Callables! - p1214](https://wg21.link/p1214)

⚔️ - ️**Exactly 2/3rds majority, but held back in EWGI for another round**

This paper was seen before about 2 years ago, and then some 6ish years ago before that. It made it through EWGI with the exact definition of a 2/3rds majority. But, as the chair gets to decide what is consensus, JF Bastien said it's alright but needs to come back to EWGI with additional motivation and examples so this does not end up back in EWG to die a death it has already died twice.

So, I'm rewriting the paper to be a huge cleanup of callables and `std::overload` for C++! It'll take some work but I think making a holistic package of things that clean up the standard's way of handling functions and disparate things that should be put into overload is a good idea for this paper.

### [Make char16_t/char32_t literals be UTF16/UTF32 - p1041](https://wg21.link/p1041)

🛡️ - **Well-championed, moved forward to EWG!**

An SG16 paper by the legendary "Robot" -- author of nonius + libogonek and the creator of the Rule of Zero -- this one was easy to get through. I had a bit of a presentation going but Tom Honermann -- chair of SG16 -- made [a much better presentation about it](/vendor/future_cxx/papers/presentations/p1041r1.pdf) then I did. I realized something as Tom did his presentation in EWGI: when you are arguing for things that contain mostly wording or legalese fixes, people are much more likely to defer to the "expert" on the wording in the room and very many people drop right out of having hyper-opinionated decision. This is especially the case with encodings and how they are worded in the standard.

As Tom presented the wording and other things, consensus to move this forward was pretty much guaranteed as people's eyes glazed over from being pointed out these differences in wording. I have a feeling this will go well in EWG since we also have data showing we have not come across a compiler that violates the UTF16 / UTF32 intention to date. This is a small paper that can make it into C++20, and it should so SG16 doesn't have to handle some crazy assumption someone comes up with about the encoding scheme for UTF16 or UTF32. Right now the industry is with us in that nobody's done anything looney with the wiggle-wording afforded them. We want to close this out before somebody creates some looney 16-bit or 32-bit encoding and tries to standardize it. It would also be great for the rest of SG16's work.


### [Named Escape Sequences - p1097](https://wg21.link/p1097)

🛡️ - **Well-championed, moved forward to EWG!**

```
const auto& ohm = "\N{OHM SIGN}";
```

Another SG16 paper written by the same legendary "Robot", this one was easy to get through. Both Tom and I had made presentations for this one, but eventually decided to [go ahead with my presentation](/vendor/future_cxx/papers/presentations/d1097.pdf) after a few tweaks. This was voted forward to EWG with strong consensus, with a few additional polls about how exactly we wanted to do Named Escape Sequence matching. The presentation shows some of the different matching forms (with some SG16 bias as to which ones to pick). The polls indicated that case insensitive matching for names was preferred among the 3 alternatives (no transformations, case-insensitive, and then full UAX #44-LM2).

It was noted during the conversation that [full UAX #44-LM2](https://www.unicode.org/reports/tr44/#UAX44-LM2) would allow for the smallest possible trie to contain all the Named Character Escape Sequence data. Note that the data would only be kept around for the compiler, not for compiled executables: these sequences are just for the compiler to insert the sequence of encoded bytes. Something that came up was the idea that having to have this data with the compiler would add gigabytes of Unicode data to every compiler.

The full Unicode Character Database (UCD) in Extra-Lazy-No-Space-Savings-Just-Make-Lots-Of-Arrays form is 10.2 MB in a DLL compiled with Visual Studio 2017. This includes literally everything all of the Unihan and Historical character data that next-to-nobody would ever need.

Name Alias and Named Sequences are only about 2 MB, with no space saving storage schemes or optimizations applied.

With luck, this can be quietly moved into C++20. It's not really worth much debate, except maybe asking for UAX #44-LM2 to give implementers a chance to crush those lookup tries down even more.

Hilariously, I spelled one of the sequences wrong on one of the slides and someone pointed out that I spelled it wrong. We got a good laugh saying that it would be a compiler error rather than just getting a bad code point because I used a valid-but-not-quite-right `\UXXXX` escape. Sometimes, even mistakes can help point out the utility of a feature, hah!


### [std::embed - p1040](/vendor/future_cxx/papers/d1040.html)

⚔️ - **Under contention because of perceived requirements.**

```
constexpr std::span<const coefficient> coefficients = std::embed<coefficient>("coeffs.bin");
// ...
```

The good news about this contention is that it actually has nothing to do with the quality of `std::embed` itself. It is wildly agreed that this functionality is wanted and needed and to-date is the #3 highest upvoted proposal request on Reddit (behind only Niall Douglas's iostreams v2 Study Group proposal and Herb Sutter's Deterministic Exceptions proposal).

But the build system and modules are in the way.

Modules make the promise that dependency discovery and module blob connecting can be done simply and easily by the compiler and a build tool without requiring that same build tool to invoke the compiler. This is fine, except `std::embed()` is a `consteval` function that runs during the typical `constexpr` evaluation time. That means Full Semantic Analysis™ must be performed to truly know the final values getting shoved into `std::embed` (potentially, you can shortcut it in many cases, such as with string literals). This makes many people sad because they want their precious (meta-)build system to be effortlessly and seamlessly told about the dependencies without having to run a compilation.

This makes my life so, so, **SO** difficult.

Knocking down on my door now are several build engineers. I now have to answer their questions. It is frustrating that this feature gets perpetually stuck with new and interesting problems. Either way, I had to write a whole new paper -- [p1130](/vendor/future_cxx/papers/d1130.html) -- to solve the problem. I have some early feedback on it from the SG15 Tooling mailing list, so I need to fix up some of the questions posted in those threads.

EWGI forwarded not the paper, but the question of "dependency management??" to EWG. Then it has to have another round trip back to the Incubator... which is starting to tire me out. This was the very first paper I wrote. I picked it because it was supposed to be the braindead-simple paper to get specified and through the committee... but, well.

Modules just had to go and complicate everything.


### [[[nodiscard("should have a reason")]] - p1301](https://wg21.link/p1301)

🛡️ 🛡️ 🛡️ - **EASIEST paper of my life, forwarded to EWG!**

Simple paper. Universally loved. People had a hard time finding reasons to actually argue against it. Finally, something that did not take a colossal amount of effort...!

Forwarded to EWG, so long as we stop by LWG/LEWG and have them give us a short 5 minute session about how much they want this. (I might just send an e-mail out on the reflector to see if we get some resounding support in that fashion, so we can avoid having to take up face-to-face Committee time with something that 90% of people I talked to said they would LOVE it.)

# LEWGI / LEWG Battle Standings

### [std::out_ptr - p1132r2](/vendor/future_cxx/papers/d1132.html)

🛡️ - **Strong Consensus to move forward, Recommended C++23 but C++20-possible with effort**

`std::out_ptr` is possibly the best thing I've ever designed. It's small, simple, easy to implement, easier to optimize, and represented an insanely widespread industry practice that _other_ people in the industry did not know about and got so excited seeing that they actually send e-mails on mailing lists about it. That's a pretty dope feeling.

[The presentation](/vendor/future_cxx/papers/presentations/d1132.pdf) went well in LEWGI. Some things needed patching up, which I managed to fix up for the post-San Diego mailing (the papers were due November 26th, so I sent all mine in). A lot of questions in LEWG around whether or not we want to solve this just for the C++ Standard Library pointers, or if we want to solve it for all smart pointers. A non-negligible amount of people wanted to solve this for just the Standard Library, but a lot more votes were for solving this for everyone. I am very much strongly in favor of solving this for everyone: having other people reinvent the wheel is not to our benefit as a community.

I also found some more usages of this in the wild. Asides from Microsoft's `WRL::ComPtrRef`, quite a few company's internal versions of this, and my own version of this, there is also one in Adobe's Chromium. It's even more widespread than I initially find, and more and more people are very much excited about this.

One of the things LEWG asked me to explore was different extension mechanisms. I evaluated quite a few and got back with their results quite swiftly in the next revision of the paper. One of the biggest contenders to get rid of the current class-based overriding mechanism was to use ADL. While attractive at first, it is actually a flamingly bad idea. Particularly, because of the way the factory functions and constructors for these types would work:

```
template <typename Smart, typename... Args>
auto out_ptr( Smart& s, Args&& ... args ) {
	// ...          ^-- This stuff right here
	// ...
}
```

Unconstrained variadic forwarding arguments are the worst thing to throw into an unconstrained, ADL-dependent overload set. The standard version would need this so it can interoperate with `std::unique_ptr`, `std::shared_ptr` and [the upcoming `std::retain_ptr`](https://wg21.link/p0468), all of which have several different flavors of their `.reset()` functions (that, or we commit to writing non-variadic versions that mimic the constructors / `.reset()` calls of all of these types...). Then, users adding their own overload to their namespaces would be caught up in the hell of trying to make sure the above function doesn't compete with their own.

This is not a nightmare I am willing to sell to the standard, so the `std::out_ptr_t` and `std::inout_ptr_t` extension points still seem like the best option.

The cool thing about this proposal's extensibility is that it comes with out-of-the-box support for [`unique_resource` and friends](https://wg21.link/p0052). This is part of the appeal of `std::out_ptr` and its counterpart `std::inout_ptr`: things that are well-behaved can participate in the standard's niceties without requiring an additional paper per ownership/resource wrapper we come up with. Simple, effective, and extensible designs built on foundational concepts like RAII help us create abstractions that can stand up to a diverse range of use cases without having to rewrite or append to the C++ specification.

Being able to talk nicely to C APIs is one of C++'s strong suits. This proposal enhances that, and if I go to the upcoming Kona meeting this is priority 0.

As part of preparing, some of the folks I met at an impromptu Future of Boost Dinner (Glen Fernandes, Michael Caisse, Zach Laine, Jon Kalb and others!) recommended I try to put this into Boost or somewhere else. I'm pleased to say that some work is [being done in that direction](https://github.com/boostorg/smart_ptr/issues/56)! I even got to drill down into some of the [performance metrics and expand my benchmarks quite a bit too](https://github.com/boostorg/smart_ptr/issues/56#issuecomment-441104027), to observe some differences in behaviors and usages of the abstraction. So instead of just having the specification and my floating public implementation (which still doesn't have a license because I'm doing a million things at once and aaaah), we can have a spicy boost implementation, yay!

All in all, I hope everything pans out nicely.

### [Optional - p1175r0](https://wg21.link/p1175r0)

🗡️ - **No Consensus to Move Forward for C++20**

The simplest optional possible to enter C++20 failed to receive enough consensus to get in, but the idea in the paper itself and other ideas to fix optional are not completely dead. LEWG Chair Titus Winters makes a distinction between a paper not receiving consensus to move forward and a paper being outright rejected: p1175 was the former. This means that this paper -- in its current form, even -- or a similar solution can be looked at in time for C++23.

This means that if someone comes back and argues for it better than I did in that LEWG session, it has a chance. However, at this point I'm just insanely disappointed in both myself and the overall state of optional. It will be 2023 before we will -- officially -- have some solution to optional. But remember that this paper was not just for optional: this was a stopgap measure for `std::variant`, `std::optional`, `std::expected` and all the others. Now, all of these types are stuck in Limbo for another 5 years.

I have a lot of things to say about optional and other "wrapper" and "composite" types in the standard library, so this section will -- thankfully -- remain as short as it is in exchange for a much more deep dive at a later date.

After this paper didn't reach consensus, I was pretty sad. I retreated for a bit to the room I like most: Core Working Group. While I was there, I actually had a nice little thing happen to me. In the Immediate Functions paper being reviewed there ([p1073](https://wg21.link/p1073)), there was a typo in one of the examples. I caught it, but didn't pipe up because I was feeling like a doofus after failing to properly explain the necessary motivation for `std::optional<T&>`. But later in the session Jens Maurer -- a really great person who handles a lot of wg21 meeting logistics -- pointed out the same thing, and I was right!

My eyes are getting better and better for wording stuff. Perhaps one day I'll be able to actually participate in Core Issue Processing...?


# SG16 - Unicode Troops Assembled

Study Group 16 - Unicode met in San Diego. SG16 couldn't meet during the Rapperswil 2018 meeting, but we got one critical paper in: Update the Reference to the Unicode Standard ([p1025](https://wg21.link/p1025)). To continue our success, the long-awaited and steadfastly worked on `char8_t` - [p0482](https://wg21.link/p0482) - made it through Core and Library in San Diego! That means `char8_t` is a C++20-era **REALITY** that we can now build the rest of out text initiatives on, with a firm foundation knowing that we have a type that represents canonical unsigned UTF-8 encoded bits. We are also no longer shackled to receiving a `char*` and having to ponder if it's in execution encoding or UTF-8 or some other whack third thing: `char8_t*` can be canonically and safely -- at the language level -- assumed to be UTF-8.

We discussed the [direction paper we wrote, p1238](https://wg21.link/p1238). We also took a bunch of polls and some votes on what we want to see insofar as direction. Transcoding and normalization were at the top of the list, but one that also crept in was a `std::locale` replacement. It's going to be bonkers hard redefining locale, but... well. We sort of accepted this job when we made SG16, didn't we?

As a side note: Tom Honermann is a great guy. Compassionate, understanding, willing to put in a lot of hard work in shepherding proposals, bouncing ideas and helping proposals, it's been great working with him in SG16. I can only hope I can produce some work that makes him proud to have me as a member of SG16. (I've already started to revive my old [`text` and `text_view` types](https://github.com/ThePhD/phd/blob/master/include/phd/text/basic_text_view.hpp), though they're mostly skeletons right now...)


# Writing History

I visited the ~~elves~~ LWG briefly. I got to scribe for one of their sessions, too. I really hope the person finds the notes I left behind useful... I did my best to faithfully represent the conversation as best as I possibly could. It's a bit challenging to scribe: I only did it once before for one whole meeting of SG16 (Tom back-filled the gaps in my work). For a lot of the stuff I presented, Ben Craig scribed. And, apparently he scribed like 15+ papers, which is HUGE! I definitely don't know how he managed to do that, I was pretty zoinked after doing it one and a half times...! Ashley Hedberg also scribed quite a few of the LEWG papers too, I think. Which was good, because her scribed notes were pretty good! I hope to get mine to Ben and Ashley's level at some point.

Paul McKenney -- a long-time engineer that has a vast swath of knowledge that I had a hard time grasping in full while talking to him at the meeting -- said I did a pretty good job with my scribed notes! So that was good.


# So Where Next?

Well. The Post-meeting Mailing is going to be coming out. I dropped a lot of my new features into that mailing, including the Flexible Array Members and Explicit Return Types for (Implicit) Conversions papers. I have no expectations that anything except Literal Suffixes and `std::out_ptr` will make it for C++20: sorry to everyone who wanted `std::embed` before C++23. It's just not going to happen until I tackle this abominable dependency problem...! I started the process of interviewing for at least 5 different companies, and e-mailed many more, for a 2019 Summer internship. I am really doing my best to work hard and keep pushing forward.

I should take a moment to talk about San Diego. San Diego is actually REALLY pretty! When I first got there I was taken on a bit of a tour by a very awesome guy by the name of Jerry Coffin. He told me a lot about San Diego and the surrounding area, how the seasons play out close to and far-away from the Sea, and a bunch of other really neat information that helped me get around on my first days. Being close to the sea was good, because it meant the temperature wasn't "BUUUUURN" levels of hot.

I also got to spend a lot of time walking around with Corentin Jabot and talking about C++ and plans and the future towards the end of my trip, and I even met Bryce Adelstein Lelbach for a dinner at the pier. There's a lot of things that will make for a complete C++, and I'm excited for it all. I feel like we can almost complete it all in time for C++30... a complete, fully functional, library-complete language...

Maybe one day when I'm not working on so many things, I'll get to relax in a place like San Diego.

There's a lot more I could've written but this report is quite lengthy already.

Thanks for reading.

Ta-ta for now! ♥

<sup>P.S.: Core is still the best room. But, I think I'm going to have to get comfortable in LWG too. And I was able to mildly follow some of Jonathan Wakely's chat in there, which means I am getting better. Soon! Soon...</sup>
