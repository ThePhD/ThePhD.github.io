---
layout: post
title: The (Late) Rapperswil Trip Report
permalink: /rapperswil-2018-c++-committee-trip-report
feature-img: "assets/img/thumbnails/RapperswilRainbow.jpg"
thumbnail: "assets/img/thumbnails/RapperswilRainbow.jpg"
tags: [C++, Unicode, text processing, future, sg-16, embed, standards, proposals, 🤝, ⌨️, 📜]
excerpt_separator: <!--more-->
---

The C++ committee meeting in Rapperswil-Jona, Switzerland was my first committee meeting, and in a foreign country no less! And, believe it or not, it went <!--more-->fantastically!

If I had all the time in the world I would sit down and write papers to improve the language in all areas! This is the kind of work I love doing: sitting down and really drilling home useful features and tools that can make developers lives easier (which is, no surprise, the entire reason I wrote sol2).

# Papers of Interest at Rapperswil

I have numerous paper ideas, but we'll focus on the ones I got to talk about, and the ones I presented, while at the HSR.

### char8_t - p0482r4

[Link](http://wg21.link/p0482)

Tom Honermann could not make it to the meeting, so being a part of Study Group 16 (SG16-Unicode), I helped present the paper. It was well received and made it through Core Review successfully! Tom actually stayed with us by speaker, thanks to Jens Maurer's amazing help in setting him up so he could voice chat live with everyone in Rapperswil. As a side note, Core is actually the most chill room on the planet. If you go to a C++ meeting, sample all the rooms, but spend a good amount of time in Core. It's fun, you learn a lot about the wording of C++, and occasionally they go slightly off track and start designing stuff. Their designs are pretty down-to-earth and grounded in reality, probably because they're Core and they have to process all the wording for it!

### Update Reference to the Unicode Standard - p1025r1

[Link](http://wg21.link/p1025).

This paper got through the ENTIRE review process in one meeting! Of course, it was a tiny paper, and ultimately we ended up updating our reference to ISO 10646, which _actually_ properly defines UTF16 and UTF32. Now we can start to use these terms normatively in the C++ standard for character encodings, which will come in handy with SG16's other foundational work such as ["char16/32_t should be specified to be UTF-16/32"](http://wg21.link/p1041) and a new, spicy upcoming paper for Named Character Escape Sequences, allowing you to use Unicode Names to specify characters rather than \U or \u escapes!

### std::embed - p1040r1

[Link](http://wg21.link/p1040)

This got a lightning round in LEWG. It is the paper I am trying to pilot through the Evolution Groups and hopefully land it in at least Core's lap in San Diego. The reception was positive but I think there's some more room for design cleanup in `std::embed`. Notably, people recommended things like adding an alignment parameter so that the data -- when loaded -- would be aligned to some proper boundary. While this is a good idea for runtime data, the desired effect of `std::embed` is to produce **compile-time** data access from a resource (e.g., a file).

If someone wants to align it at runtime to a specific byte boundary, append a null terminator to it, or do some other thing (like create an MD5 checksum or do compile-time decompression before shoveling it into the binary), then the user should write a `constexpr` algorithm to do that. Therefore, the next version of `std::embed` is likely going to tear out `embed_options` and the `alignment` parameter: these are things better left controlled by the implementation and implementation-defined `#pragma`s and other mechanisms. I will likely only add them back in if the implementation of it in clang dictates they are necessary, and after I put it in the hands of some users for a little bit.

### constexpr! (immediate functions) - p1073r0

[Link](http://wg21.link/p1073)

I had sent a message in advance letting Daveed Vandevoorde know that this paper (and `std::is_constant_evaluated()`) were sorely needed, especially for `std::embed`, and that I would help him get it through the committee by any means necessary! Thankfully, it didn't need my help at all.

Despite some minor grumblings about trying to understand what it does, once we established exactly how it worked it was pretty much agreed that we needed the ability to have a function decorator that did not have this wishy-washy "well, maybe I'm called at compile-time, maybe I'm not, ooOooOooooo!".

There's still quite a bit of bikeshed about what the final sequence of tokens / context-sensitive keywords might be, but `constexpr!` is about as good as any. I believe there might have been talk about `constfold` and the like, but nothing terribly official. I will note that `constfold` is an awesome context-sensitive keyword name!

# On the Committee Itself

If you have never gone to a Committee meeting and you have an entire week to blow with a bunch of nerds who have excellent taste in food and lots of great stories to share, you can do like I did and sit in the middle of a bunch of insanely intelligent lads and lasses and just soak it up. It's absolutely worth it and it really puts a lot of the hard work into perspective. It's also pretty amazing that you can walk up to people who are basically C++ demigods, shake their hand, and just... have a chat. Literally, have a chat, right then and there, with people like Herb Sutter or Hubert Tong.

### Voting

A few things about how voting worked and the like perplexed me in the beginning. Many people are sent not just for the greater good of C++, but to represent their company or entity. Thusly, many will very stringently vote along predefined lines for certain hot-button issues, whether or not they themselves believe it is a good idea.

This makes sense, since the vast majority of attendees are ISO members whose presence is both bankrolled by their company and also serve as ISO representatives in the US/French/German/Dutch/Netherland/Polish/etc. National Body. Voting along company lines is to be expected, but I suppose I was still surprised for the first few votes I saw out of some individuals, before in my head I went "ah, right, they belong to contingency X representing Y".

As a student bankrolling my own trips, I don't have anyone to answer to. I could put my hands in the air like I just didn't care! ... Except, well, for a lot of votes I did care. There was even one vote where I was the only person who put my hand up for it! A wise man once said:

> Remember to always speak up, always ask your question, and always raise your hand, because you **are** the last line of defense. They **must** answer your questions and concerns! - Bryce Adelstein Lelbach, 2018

And he's right: votes that pass straw polls in the rooms have to be voted on by the ISO members in plenary at the end of the week. If you did not stop it then -- and if you're not an ISO member -- the straw poll **is** the last time you can put a stop on something!

### Bikeshedding

As tends to happen in Evolution rooms like Library Evolution Working Group and Evolution Working Group, things tend to get... bikeshed. A lot. Sometimes, this is a great thing. Other times, we spend literally 2 hours drilling down on something that honestly has us pontificating in circles. It's hard to strike the balance. Especially when you're on hour 10 of discussion. Which, speaking of...

### Yes, it's Hard Work

Sometimes, there are events that force us to go outside or enjoy the locale. All the other times, though? You're in a room. Sometimes with no windows. And you're reading. And talking. And reading some more. Talking. Posturing arguments, structuring things, performing last minute paper edits, trying to do that tiny bit of work while also participating, listening in on IRC for if a room you're not in pings for your presence... But at least you have a break at lunch and dinner time, right?

Wrong.

You're going to eat with the same people you were just having a discussion with. Guess what you spend most of dinner discussing? That's right: exactly what you have been discussing, especially if it was a S P I C Y topic. You get there at 8h00 in the morning and jump in nearly immediately. You get back at 22h00... if you're lucky. I had to Super Solid Snake into my AirBnB past midnight and do my best not to wake my host several times. These are longer and harder and more mentally taxing than even the most stringent work week, especially since you quite literally do it for 6 or 7 days in a row. It's fun, but it's absolutely exhausting! And even when you spend literally 14+ hours listening and talking and listening and talking some more and convincing and voting and passionately appealing and stalwartly defending and being the greatest paper champion you can be...!

### It's not Enough

Hot diggity, is it not enough. So much to do, so little time! And sometimes, you don't even want to just talk about what's on the agenda: there are C++ legends in the room, of COURSE you might have a nerd moment and totally steal someone for lunch and just pick their brain. A lot. So much so that you might as well have an ice cream scooper rather than a pair of chopsticks for your picking, really.

Of course, this is why I've already booked my trip to San Diego nice and E A R L Y. I've gotta get me some of that premium Committee Time™, especially with the new things I have in the works. It's sort of necessary I get ahead of the curve, because goodness gracious does the C++ Standard need a lot of help. But that is fine, because we're all here to help it grow into something we can use for the next 30 years successfully!

# Oh, the Groups

Right. The committee is currently split into four factions: ~~Orcs, Elves, Humans and Dwarves~~ Evolution, Library, Core, and Library Evolution. The Evolution groups are the gatekeepers that hammer out design and feature set of things related to the language and the library, respectively. The remaining two (Core and Library) are for the exact words that end up in the standard (again, language and library respectively).

There are also Study Groups, but maybe those can be reviewed later. The bigger groups have private e-mail "reflectors" to keep a high signal-noise ratio as high as possible and to not let any old person come in [and throw a massive tantrum](https://groups.google.com/a/isocpp.org/forum/#!topic/std-proposals/qd3L1-bGg1A).

### Core

As I mentioned before, Core (or Core Working Group) is basically the maximum chill room with the best people. 10/10 cannot recommended highly enough. Even if you find it boring they are still the most down to earth lads and lasses you will ever meet, and also the most pedantic! When they do slip a little and go into design land, their ideas are **fascinating**! Sometimes, I feel like Core should take a day off and just be EWG #2... but that's a pipe dream that'll never happen. Thankfully, Core members occasionally "defect" out of Core to go sit in EWG or -- more rarely -- LEWG. They always boomerang back around, and its back to making sure the crazy proposals EWG forwards through don't shatter the language into pieces as specification is hammered out.

### Evolution

Evolution (or Evolution Working Group) is the biggest group. Most people sit here, because who doesn't want to gatekeep all the spicy new language features? I find this room the most tiring to sit in and debate: with so many people, there's bound to be some serious investment in new features and new ideas. People who want to fix the language come here. People who want to add new features come here. People who want to propose breaking changes come here. A high volume of papers pass through here. Ville Voutilainen did a good job keeping everyone on track in Rapperswil, most of the time!

Core is still better, but if you're looking for trial-by-fire submit your first proposal here. If you don't do your homework, I can guarantee you will get cooked reeeeal quick in EWG. Nicely, but you'll still be cooked! So make sure your proposal is polished and ready, preferably dropped on reddit or in std-proposals or somewhere for somebody to take a first gander at it and kick the thing around a few times.

### Library Evolution

Library Evolution (or LEWG, which is just 2 more words away from LEWGIE -- still haven't thought of the words yet) is another fun room. Libraries come here to be bathed in the cleansing waters of good design, or drown in their misdeeds. With a room full of people who have written their own third-party libraries and a handful of older boost contributors at times, this is the friendliest place to go if you are looking to learn something about design... and get an earful of opinionated banter about nested namespaces and API surface area. This room gets to the meat of topics very quickly, and chews on it thoroughly: even if your library or utility doesn't pass LEWG, the feedback is immensely valuable every time.

This is where `std::embed` landed first in a lightning round (basically, "we have 20 minutes before lunch who wants to do their paper with all due expedience?"). Ideas I did not even think of, nor were brought to my attention in all the prior discussion, came up here. Thankfully, it was just little things I missed and none were major design flaws.

This is also where big-ticket library features get discussed as well. Executors and Networking, for example, go through here. A lot of interested parties come to the table when these things show up. Ranges were also chewed through here in Rapperswil. I will note now, however, that you should not just submit your favorite library here. It needs to be mulled over carefully. It needs to be well-specified. You need to consider extensibility, performance, maintenance and how much it does or does not play nice with the rest of the standard where applicable.

### Library

The only room I did not expose myself to enough was Library. Library is responsible for wording. You hand them a feature, they make it specification. I sat in one half-session, and one library issues processing. I could only offer one meager comment in the issues processing session, and whisper a few things to the amazing C++ veteran Walter Brown about a few ideas I had. As explained to me, this room is 100% the bottleneck for most standards features. Prioritization and off-shelf meetings between members help process things at a rate acceptable to the explosive growth of the language.

If the C++ language is big, the C++ library is huge by comparison, and there's quite literally tons of issues and only a handful of experts willing to not only implement them but wrangle the wording to be consistently good. This is the room I want to help so desperately, but I feel completely outmatched. From Jonathan Wakely to Marshall Clow to Billy O'Neal, this is quite literally the "Rocket Scientist" room. The little knowledge I have is utterly eclipsed by just a fraction of their expertise and its daunting to want to make suggestions about wording when I understand nearly nothing of it.

Thankfully, Marshall Clow sends out weekly "let's fix this issue" or "let's read this paper" e-mails to the Library group, and getting on that list has taught me a thing or two (or twenty, if I'm honest). I'm still not confident enough to chime in about issues processing, but I have started writing the wording for my paper and this will steadily allow me to reach a place where I can confidently help. It is going to be a **long** time out, but I'm persistent!

# What Now?

Well, the post-Rapperswil mailing is out, CppCon is coming up (of which I've signed up to be a volunteer and also a speaker. I clearly didn't learn my lesson from overburdening myself last time), and I have some papers I need to get in for San Diego.

I have much more lighthearted papers like [p1132 - out_ptr](http://wg21.link/p1132) to keep me afloat. I even managed to get wording written for [the upcoming r1](https://rawgit.com/ThePhD/out_ptr/master/papers/d1132.html), even though my wording is essentially trash-fire material right now. Small, iterative improvements, right?


That's all for now. Apologies for the lack of posts; there'll be more coming in the next few weeks!
