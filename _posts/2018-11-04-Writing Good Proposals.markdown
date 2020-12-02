---
layout: post
title: 📝👍 - How to Write a Good Proposal to C++
permalink: /writing-good-proposals
feature-img: "assets/img/pexels/businessman-approve.jpg"
thumbnail: "assets/img/pexels/businessman-approve.jpg"
tags: [C++, standards, proposals, writing, 📜, 📝]
excerpt_separator: <!--more-->
---

I am not great at writing proposals. In fact, I am absolute 🐕 💩 in many aspects. But, I had feedback and help, so I wanted to put out some tips so that when people write proposals they are much less trash when they hit the Committee. I want to read 🔥 💯 ✌️ papers when they hit the Committee, rather than being the fuel _for_ a trash-fire.<!--more-->

# Survey the Area/Industry/Academia/Hobby Code

Seriously. [p1175](https://wg21.link/p1175) can get as far as it does not because it tries to take my little library or idea and cram it into the Standard. It surveys the industry and many codebases. "Monadic operations is the #1 requested feature" from my [last article](/sandiego-2018-paper-review-II) did not come out of thin air: it came out of months of research and running a survey and doing the legwork. This is especially critical for Library code. How many people are using this thing being proposed? Are there multiple implementations? Are they high quality or poor quality? For example, having _lots_ of available but low-quality implementations means that maybe a type/function has utility but needs a careful hand to make it work out. Having multiple high-quality implementations means the type has seen great iteration and success, but suffers from not having a common spelling. Most importantly,

_History_ teaches us what people needed and what mistakes they made.

It does not always teach us why, but identifying that and writing it out into a Motivation section is going to be part of the job. We have lucked out, however: code archaeology does not require dusting off bones and performing carbon dating. The people who wrote the first reference wrappers in C++ are still alive, and they will answer e-mails! (Most of the time: sometimes their e-mails are outdated and then it's no fun.) The people who wrote the first STL are still alive and are giving talks. Scott Meyer's hair is just as fabulous as it has ever been, and he answered all my questions and more when I saw him at CppCon.

Grab a shovel and dig.

Sometimes, tons of gold is struck: [std::embed](/_vendor/future_cxx/papers/d1040.html) and [std::out_ptr](/_vendor/future_cxx/papers/d1132.html) are examples of this. Both of these have tons of prior experience in various different forms. Standardizing them is (going to be) easy because there is a huge momentum and need for such tools in the standard. Figuring out the minutiae will be part of the job but fret not: if it's standard practice its less about design and more about just making sure the final result crafted fits the intended needs. Remember that covering 90% of uses is fairly good.

# There Need Not Be Wording

Revision 0 (r0) proposals can have wording written later. It is important to properly state the proposed change, fully provide motivation and history, show benchmarks and other relevant data if it is important, and carefully document the design and rationale of your solution. I deleted my first set of wording out of [p1132](https://wg21.link/p1132r0) for r0 before sending the paper in its first mailing. And still, it was [praised as one of the best r0 papers someone had ever read](https://twitter.com/AlisdairMered/status/1014937023520624642)! If I can take a moment... <sup>\<fandork>SQUEEE, Alisdair Meredith AND Peter Dimov liked somETHING <b>I</b> WROTE hafhkawlfauil SQUEEE!!\</fandork></sup>
 
_Ahem._

Solid r0 papers make it easy to get motivation for a feature before you put your heart and soul into the wording. And when Revision 1 comes out and the wording is written, just be prepared for a little truth:

# It Will Suck

Some people are very pedantic and particular, care very much about the language used, and catch onto this stuff very quickly. Or, they have been around for a long while, have implemented these things, and have been baptized in the holy fire of long-term compiler/library development. They typically have the scars, triage experience, and bug reports to prove it. Chances are,

wording is going to be 🐎 💩 on the first writeup.

Ask someone for help. There are people who will help. If the feature is not big then other people may not be necessary: look at the Standard or other proposals (the latest revision, preferably). Look at the wording. It will help.

# Help Them Help You - State the Intent

Oftentimes, I noticed something in LWG with Marshal Clow, Alisdair Meredith, Dietmar Kühl, Jonathan Wakely and a few others present. For nearly every proposal there was lots of work on and for every issue that was processed relevant to a recent paper, one question was **repeatedly** stated, almost ad-nauseum:

> Was that the intent?

Was that the intent? Was that the author's intent? Did they mean this? Is that what they were going for?

I can write my whole proposal out and I'm sure anyone can clobber together the _full_ feature from reading the entire thing, but let's be perfectly clear: some of these papers have specifications in the 100s of pages. ~~The High Elves~~ LWG has the smallest headcount of pretty much any room, save for the smaller Study Groups. They do not have time to read the entirety of my paper and **piece** together my motivation over pages and paragraphs of prose.

So, for my papers I did the following: every one that has wording comes with a section above the wording named "Intent". This means that -- without having to read the whole paper -- LWG members can just **read the intent section** and know what I'm trying to say in "Standardese". The intent is usually provided in bullet-list form in English or pseudo-Standardese, describing behavior and effects regularly without the formalism of the standard. And if that doesn't match, rather than having a 3+ e-mail back-and-forth with me or having my paper champion have to sit there and say "well gee I don't know what they meant I'll have to go back and ask the original author", LWG can instead just send me a list of corrections/notes about how wrong I am, and I can go fix it. Or, they can tell me the intent is wrong because it is unimplementable/unspecifiable/hot garbage. This saves not only LWG precious review minutes, but also saves me time! 🎊 E v e r y b o d y 🎉 w i n s ! 🎊

It also helps me refine my wording so that I begin to associate certain intentions with certain wording patterns. Essentially, I am teaching myself how to "git gud", as the young whippersnappers might put it.

That's all for now. Please, make sure to write good proposals. It is necessary! If you think your proposal is not good enough, then hold onto it until its better. I submitted a number of proposals, but I held back about 4 proposals for C++20, and _that's okay_. It is better to ship good, right specification than garbage we need to keep patching over for 20 years!

Ta-ta 'till later. ♥

P.S.: Have a goddamn Tony Table! (Before/After tables, but we call them Tony Tables in the Committee. There's some exemplary ones [here](https://github.com/tvaneerd/cpp20_in_TTs).)
