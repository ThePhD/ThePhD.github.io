---
layout: post
title: A Perspective on C++ Standardization in 2018
permalink: /perspective-standardization-in-2018
feature-img: "assets/img/pexels/meeting.jpg"
thumbnail: "assets/img/pexels/meeting.jpg"
tags: [C++, standardization, 🤔]
excerpt_separator: <!--more-->
---

This is what I get for opening my big mouth in Twitter threads with people far my senior. Let's just hope I don't get in trouble, here...!<!--more-->

I ended up getting pinged in a thread that was going on about [Eric Niebler's ranges post and general C++ lamentations](https://twitter.com/Timewatcher/status/1078222716442800128). From that, spawned a Blog Post from [Game Industry Legend Aras Pranckevičius about their lamentations for C++](http://aras-p.info/blog/2018/12/28/Modern-C-Lamentations/). In it, a lot of C++'s well-known pain points were once more brought to the forefront when discussing the upcoming Ranges: terrible build/compile times, cognitive load to understand new (or revamped) features, put-it-in-a-library fetishism that often tanks runtime performance except when inlined by the compiler, and more.

I also got asked a question:

> One of the arguments in this recent storm has been that many proposals going into the committee come from people "outside the trenches", with insufficient real-life, heavy production C++ experience. Do you think this fits your profile? I would be interested in your point of view – [Javier Arevalo, December 28th, 2018](https://twitter.com/TheJare/status/1078725400048685058)

The short answer: no, because I'm not proposing things "as a student", nor am I trying to get what are really my own ideas into C++ (well, mostly. A few of my sneaky sneaky hush hush papers are pushing my pet ideas into the library and language, but _shh_!).


# The Long Answer

Most of my papers are "I have observed a problem that crosses code bases, libraries and industries: that's probably something worth putting into the Standard then, no?". That's my first litmus test for trying to put something in the Standard. If the problem happens to more than just me or my code base or my library or my industry, then it is a good idea to fling the paper out to the wider C++ community. All of my papers have had three or more people in different industries describe to me a problem that could be solved in such a fashion: in that way, rather than just solving my problem and having a flimsy paper backed only by my plucky, young student conviction, it has real roots that can resonate with other people.

I have seen it happen to other papers which do not quite do this, or which do not fully consider their merit or usefulness in the global ecosystem. The paper is typically shot down or discouraged because it is a questionable idea (or at least a good chunk of people find it questionable) and it has no implementation experience to refute people's doubts from a publicly available distribution.

Implementation experience can be important, especially if the feature is easy enough to implement as a library and has wide-applicability. For example, [p1132 std::out_ptr](/vendor/future_cxx/papers/d1132.html) had a marginal number of weakly-for, neutral and against votes because they wanted to see more publicly-available implementation (in Boost, on Github, whatever). I had one but it needed "more stars" (for my Github implementation). Still, the paper was able to pass the general bar for consensus because I could walk people through both the wording and implementation, provide benchmarks, and had people in the room who saw the abstraction who said "this is exactly useful to things we have done in the past, I want to see this".

Note that formally the C++ Committee in general does not require an implementation of a language feature (because not everyone is a compiler engineer) or a library (because that could be prohibitive of just good ideas in general), but a fair amount of people who attend will ask that individuals do bring an implementation of their idea to help answer questions they have.

It is also important that the C++ Committee **does not ship an implementation**. I can throw up an implementation of some idea as much as I want to, but the implementation only helps answer questions. For C++, you need an ironclad specification with wording that will eventually go into the Standard, and this is where the disconnect happens for a lot of folk. You can roll your fantastic thing in your engine / application / middleware / scientific package? Awesome!

Now write a specification for it.


### The Rigor of Standardization

[range-v3](https://github.com/ericniebler/range-v3) is an implementation of the Ranges Support library we will get in the standard (mostly, some things need to shift). Victor Zverovich's [fmt](https://github.com/fmtlib/fmt) is an implementation of what is going to be in the standard library soon as well. Neither of these libraries get dumped into the standard as-is. They need to work hard to take their ideas, their implementations, separate out the details, hammer out the specification, and write the wording for it. Other people will implement it that are not Eric Niebler or Victor Zverovich.

Can their specification stand up to someone picking at every letter?

This is where most people's enthusiasm and proposals "fall on the floor" or "die". The idea is a good idea in your own application or code base, but now you're trying to prove its useful for _everybody_. Can you prove that, really? _Is_ it good for everyone? An implementation helps convince people that it is good for everybody, and that they just need to work hard on making sure the wording is iron-clad. Surviving this back and forth through Design Review in LEWG or Feature Review in EWG is rough. Some people survive this process, making it to the Wording Groups, and then get extraordinarily burnt out when someone brings up a concern that was not brought up for the 2 or 3 meetings (nearly a year) of work you have been doing.

In fact, in the case of [Literal Suffixes for size_t and ptrdiff_t](/vendor/future_cxx/papers/d0330.html), the original author actually went through the entire Standardization Process, from initial paper to final wording in LWG. At the last minute, LWG shook their heads and say "this belongs as a language feature", and sent them back to the start of the Standardization Pipeline at EWG. This is soul-crushing for motivation: there is an argument here for whether or not the Committee failed the original author before I picked up the paper in not sending them to the Language to begin with before he had committed to writing wording, but mistakes do happen and ceremony does get repeated. Ad nauseum.


### Surviving the Process

The process can and does burn people out, even [grizzled veterans](https://www.reddit.com/r/cpp/comments/a2o265/the_san_diego_2018_aftermath/eb38yor/?context=8&depth=9). While I understand that mistakes put into the Standards ink are forever, I think that in many cases getting ideas in front of people (the right people) and getting a healthy dialogue going is incredibly hard with C++ Standardization. I've sent out a number of e-mails on the internal reflector that received -- quite literally -- 0 replies and deafening silence, even after confirming the interested Committee Group received the e-mail on that internal reflector.

I personally have my own process for dealing with such things. All of the e-mails I have sent to individuals directly get answered, but it's hard writing 12 different e-mails for something that should only be said once. It is also draining to send group e-mails, only to find out you might have tripped some sort of live-wire of a discussion and are terrified to send another e-mail because is it rude or wrong to consistently poke 100+ people for a response?? I'm just a plebeian student, and I'm doing my best, does that silence mean strong agreement or is it just another timed trip mine that I set off and the countdown timer is just ticking off the seconds to the Face-to-Face in Kona?! Ultimately, you just decide to drop it because its kind of ridiculous to try and individually ping and cater to the 20+ people you want an opinion out of, who you _know_ are going to be in the room and slam-dunk their concerns in the actual face-to-face meeting and could this have not been done literally 3 months in advance so I'm not sent back to the drawing board for another 3 months...!

Bleh.

Silliness aside, the process is grueling. And sometimes there are invisible rules. It's also hard to coordinate conversation with both the outside and the inside of the Committee. I have ideas on how to solve this, but I am going back to school and developing such tools will not be something I will have time for until the Summer kicks off.

Not everything is like this, though. I know personally I'm picking up papers I see dropped on the floor and advocating for them. (Literal Suffixes, SG16 papers, etc.). I also know Bryce Adelstein Lelbach has done a tremendous amount of work in finding what were once un-championed, Graveyard-destined papers and pushing them back into the Committee. Corentin Jabot also picked up parts of the motherload of all Library Graveyards -- the Library Fundamentals TS v2 -- and moved some things out of it towards standardization like `source_location` and friends.

There are probably other people working on it. We have finite abilities and finite time, though, so if you can help I would certainly appreciate it! Of course, I am just a student and not a P-Member National Body constituent; the best I can do is represent your paper to the Committee when I am there and vote in straw polls!

Which brings me to another point...


# The Composition of the C++ Standardization Committee

To be brief, currently the C++ standardization committee is populated by people who usually come from companies who have the build infrastructure, the tooling support, and the means that fit their goals. Here's some of the companies/groups I know that show up at the C++ Standardization meeting that pop into my view a lot:

- Bloomberg
- Microsoft
- Google
- IBM
- Sandia National Laboratories
- British Standards Institute
- Synopsis/Coverity
- Apple
- Facebook
- NVidia
- Qualcomm
- Qt
- VMWare
- Codeplay
- the Boost Community

There's something about most of the people from these companies that participates, though. Most of them are there as compiler people. Plenty of them do or help scientific computing. A few people work with game developers, but they're mostly "related to", not "do it as a job". Low-latency financial players are also not in much of a majority here, and I do not even remember the companies the few that do.

This means that the typical concerns in low-latency or interactivity communities might not be seen, especially since "pay for me to attend a C++ meeting a few times a year" can be prohibitive, especially when those companies are already footing enormous bills to be part of groups like Khronos (which is a much better value proposition for them, if I were to hazard a guess?). Not to mention that the feedback loop for features is probably much more compelling in groups like Khronos or at annual conferences like GDC than the C++ Standardization Committee: direct access to new hardware and the people developing in, guaranteed implementation -- buggy or not -- that comes before full documentation and specification means that even if somebody is not well-versed in every corner of the spec, they get to play with and work on and provide feedback for ideas. This is very different from C++, which ships a specification that will maybe see larger adoption in 1 (at best) to 6 (at worst) years?

To add to that, in general the C++ Committee is populated by people who have large infrastructures and who can just throw a build farm at the problem and help their code continue to scale up, even if it is not immediately wrangle-able by a singular developer on a single machine. They are people who have throughput concerns that rank much higher than individual latency concerns (because individual latency concerns can be hired / build farm'd away). If you want people who have concerns more aligned with your concerns about any feature or subset of the language,

bring those people to the C++ Committee or ping people who you know are going to represent your concerns!

There is a woeful lack of people whose _primary_ concerns are compilation times, debuggability, or interactivity (other people have those concerns but given where they work and what they do, it is not at the top of the list of issues). I know this because I had many of these concerns, but at the moment I'm still just a student of Graphics, not an industry veteran. I cannot bring the same issues and real-world pain points to bear because I ultimately do not have the same breadth of experience (hence why I am only working on things unrelated to graphics or the newly minted Linear Algebra SIG because there are actual industry titans working to help people there). I am doing my best to represent features that are not the Big Sexy features but still make working with C code and other codebases easier (`std::out_ptr`, Flexible Array Members for C++, conversion operator improvements and more). I am working on Unicode things in SG16. But I cannot work on everything.

You must bring your representatives. You need to bring your companies, your code bases. I understand that a long-term commitment to a potentially arduous committee process can be painful, especially for Game Developers whose companies are often fairly liquid and whose landscape shifts constantly. But SG14 can help you keep a standard place to funnel ideas, and certain specific people can do their best to represent you and push these concerns to the larger Committee audience. Reach out, please reach out!


# And That's It

I think it should give people some decent insight into the process, and what it means to bring a paper to the Committee and participate. Things may change, they may not: I'm certainly here for the long run, so I'll most certainly report back if things change.

This was written pretty hastily, so forgive my typos and maybe any lacking details, I'm always available to answer questions!

Toooodles. 💚
