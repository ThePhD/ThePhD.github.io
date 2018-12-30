---
layout: post
title: Getting You There - Your C++ Standardization Efforts in 2019
permalink: /getting-there-standardization-in-2019
feature-img: "assets/img/2018-12-29/aircraft-moon.jpg"
thumbnail: "assets/img/2018-12-29/aircraft-moon.jpg"
tags: [C++, standardization, logistics, ✈️, 👛]
excerpt_separator: <!--more-->
---

As part of a series of "[I open my big mouth](https://twitter.com/thephantomderp/status/1079001746402365440) and [volunteer to do more things](https://twitter.com/thephantomderp/status/1079164306137235457)", I am going to do my best to be brief and do more than just give observations about the C++ Standards Committee, but give you some of the tools on<!--more--> how to get there. This also comes in the wake of [a big conversation a lot of folk started](https://twitter.com/_Felipe/status/1078932775103803393) over on how exactly we can influence the C++ Standards Committee's direction better by getting our voices heard and being present when decisions are made.

I won't talk about [SG14](https://github.com/WG21-SG14/SG14) and such, but you should know that SG14 is the Low-Latency / Games group and they do a _lot_ of work to try and handle where a good chunk of these concerns come from. At the bottom of this post are a bunch of links to Study Groups I have personally participated in at least twice, and can direct you to some of their resources.

This post will talk about what you do after you go through some of those channels, get yourself a paper, and want to attend a C++ Standards Meeting. It unfortunately will not tell you how to secure 1 week of time off from work, how to convince your husband to handle all 3 kids while you're doing C++ work, nor will it make your boss suddenly understand that this is actually a seriously volunteer effort and yes the meeting is in Cologne, Germany near the River but it's _not a vacation for the Love of God_...!

Any who:

If you're facing Financial Hardship, are a student, are self-employed, **and** have written a proposal that the chairs of the C++ Standardization Groups (Library Evolution, Evolution, Core, Library, Parallelism/Concurrency, and similar study groups) deem necessary to help move the language forward (in large or small ways), you can apply for [Grant Assistance from the C++ Standards Foundation](https://isocpp.org/about/financial-assistance-policy). If you have an employer but that employer will not cover the full cost, you have papers to present (yours or on behalf of others) and similar, [you can apply for Travel Assistance](https://isocpp.org/about/financial-assistance-policy).

I will talk about Travel Assistance, because that is what I have applied for and successfully received.


# What You don't know won't hurt You, But it will hurt Your Wallet!

My first C++ Trip to Rapperswil was funded entirely out of my own pocket. I committed to being locum for a number of papers, from p1025 - Update the Reference to the Unicode Standard, and others. I talk about some of it [in a trip report](https://thephd.github.io/rapperswil-2018-c++-committee-trip-report). It was fun, and I learned a lot, so I decided I should start writing papers and doing work on behalf of the C++ Community. I already had a paper out on [std::embed](/vendor/future_cxx/papers/d1040.html) at the time but was woefully unaware of the Travel Policy and basically just decided "yep I'm going to show up to this meeting" without really announcing my intentions too loudly at anyone. I increased my paper writing capacity as I came upon more and more problems to 11 formally proposed papers, 1 straggling paper I need to clean up and combine with another, and 1 new one I need to write for the pre-Kona mailing thanks to my Big Mouth.

This is a lot of effort but it ultimately comes down to time. The hard part was the financials. Bryce Adelstein Lelbach actually let me know they would help me cover the cost of getting there and back, and he clued me in for the Executors and Modules between-official-meetings Meeting that happened.

The first time I inquired about it, I ultimately could NOT use the Travel Assistance Policy (please read it in full) because I derped on what was required. This was for the pre-San Diego Modules and Executors meeting that came before CppCon 2018: I applied to have some of my costs reimbursed before I went, which required that I work out a paper that needed to be seen at that pre-San Diego Modules and Executors meeting. I actually did not have a paper for that (go me and my bad reading comprehension), and thusly was not applicable for the Travel Assistance. I still was able to attend; I foot the bill for a one-room AirBnB which -- around most U.S. cities -- is much cheaper than a hotel. I only recommended AirBnB if you're a hip, young, healthy lad/lass who can handle potentially unusual arrangements or not having lots of the usual amenities and guaranteed peace / quiet. (If you're spending most of the day out, then all you really need is a bed and bathroom anyways, right?)


# Second Time's the Charm

The second time, I did it semi-properly. I had several papers going to LEWG and EWG, I had worked out the wording, and I had spent a lot of time also preparing to present other people's papers. I sent a very detailed e-mail discussing the papers I wrote, my hopes for what they would accomplish with a short abstract of what they did, and what they were worth to a person at the Standard C++ Foundation. This was also the _wrong_ thing to do: the **proper** way is to actually message one of the group chairs (the Standard C++ Foundation keeps an extensive list of that [here](https://isocpp.org/std/the-committee)). Many of the people here are also on Twitter, reply in the [std-proposals Google Group discussion](https://groups.google.com/a/isocpp.org/forum/#!forum/std-proposals), and have e-mails you can find in some way (through that web page, through the Boost Community, from their mailing list posts in interest communities, etc.). They are all fairly active.

Send them an e-mail with your paper(s), your request, and everything necessary about it. If you need help, ping other people you know are committee members (on Reddit, on Twitter), DM them to at the very least check over your paper. Post your paper on std-proposals to put it in good shape before you send it to a Chair. Make sure you do your search: [write a good proposal](https://thephd.github.io/writing-good-proposals). They get back to you, saying "Yes, this person's work is useful and I sponsor their travel". And then that's it, it happens. Some things of note, however, that are true as of December 29th, 2018:

- Sponsorship is typically done in the form of Reimbursements, and all Reimbursements and given through tangible receipts. Save your flight receipt, taxi receipts, and everything, please;
- Travel Assistance covers up to Coach / Economy on a plane ride;
- Meals are not covered;
- and, transportation to/from the airport is also covered.

And that's it. I also like programs like these because they are based purely on something I think is a fairly good equalizer: financial hardship. There's literally no other requirements! If you can prove you are willing to do the work and write papers or pick up papers that need standardization or contribute in a positive way to the Committee, they will help you get there. Just please make your case as well as you can, and make sure you have actually written a paper!


# And that's your in!

The Standards Process is time-consuming, but it *can* be accessible to those of us not traditionally in the systems programming sphere who still want to contribute! Please be sure to do your due diligence and bring high-quality work to the C++ Standards Body. If you feel like it needs more work, [SG14](https://github.com/WG21-SG14/SG14) calls are open and free to participate in, [SG16](https://github.com/sg16-unicode/sg16) (if you're doing text) calls are open to the public, the Linear Algebra SIG coordinates through SG14 and does work, [SG15](https://cppcast.com/2018/06/titus-winters/) has a public mailing list and they will happily beat your tooling-related proposals up (they beat up [p1130](https://thephd.github.io/vendor/future_cxx/papers/d1130.html) into pretty good form for Kona), go to the [std-proposals forum](https://groups.google.com/a/isocpp.org/forum/#!forum/std-proposals), and more...!

You can do it. We can make C++ better, as a more diverse group with a more inclusive set of interests!

I believe in us! 🧡🧡
