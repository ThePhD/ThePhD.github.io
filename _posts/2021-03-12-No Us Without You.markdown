---
layout: post
title: No Us Without You - elifdef and elifndef
permalink: /no-us-without-you-elifdef-elifndef-c-n2645
feature-img: "/assets/img/2021/03/pinky-promise.jpg"
thumbnail: "/assets/img/2021/03/pinky-promise.jpg"
tags: [C, Standard, 🎉]
excerpt_separator: <!--more-->
---

The C Meeting has ended officially as of like less than 2 hours ago. I will write all about what happened in a later post, but I wanted to talk specifically<!--more--> about something that happened on Monday, during the meeting, with [proposal n2645](https://www.open-std.org/jtc1/sc22/wg14/www/docs/n2645.pdf).

N2654 is Melanie Blower's paper for `#elifdef` and `#elifndef`, a long-standing hole in the preprocessor and a constant consistency footgun. Even I experienced this pain, and I was grateful to see Melanie put it forward. We don't often get to do simple things, so Melanie is doing the Lord's Work, here.

I took to Twitter on Monday and wanted to shout OUT to the very FIRMAMENT of HEAVEN about what happened, but ISO and its various sub/working-groups have rules. So I specifically can't reveal anything until the actual adjournment of the meeting. But the meeting is over now so, let's get right to it:




# The Paper Was Accepted!

"...Wow, that's it? Seriously?? You could have just tweeted that and then went along your merry way???"

Well, yes, and I will tweet out some things (or I have already, by the time you read this?)! But, that's not what I wanted to talk about. I wanted to talk about the remarkable way in which this paper was accepted during the meeting.

See, we were discussing the paper. All the usual arguments were brought up for and against it:

- "it's a tiny feature";
- "I literally implemented this feature during the break in 10 minutes, implementability is not a problem";
- "it improves orthogonality/consistency with the language";
- "but do we really need it?";
- "`#if defined(...)` does all the same things and more generally, why do we need these shortcuts?";
- "if I had a time machine, I'd blow `#ifdef` up and just make people used `#if defined(...)`, it's better and more flexible and represents the same functionality";

and so on, so forth. A slew of folk made the exact same arguments through various other communication mediums when they saw the paper; there's not that much that's new under the sun, after all! But, the longer the discussion went on, the more it turned into this not being compelling enough to add, and that `#if defined(...)` covered the functionality well enough. It felt like it was going to be a pretty narrowly close vote on the No side. I spoke up, talking about how out of all the features I've done work on, this one actually got the most messages for a C feature (not C++ ones). (It even beat out `typeof`, don't ask me how this is more popular than that!)

In mentioning that some folk had indeed messaged me directly about [n2645](https://www.open-std.org/jtc1/sc22/wg14/www/docs/n2645.pdf), someone also got to share one of the e-mails of support they received from a random Senior Embedded Engineer directly with the whole Committee. And, once they shared it...



# Everything Turned Around

The objections dropped away. We took a vote; the paper passed.

A lot of the time, our end users — the various C and C++ developers, the millions of you worldwide trying to tamp down bugs and deliver features to customers, hobbyists, modders and other people — feel like we're not listening. Some might even accuse us of having [Big Huge Designer Brain](https://twitter.com/pcwalton/status/1367966466956431361) and getting into a silo.

And you'd not be wrong!

Some of us were, very much, going to exhibit Big Huge Designer Brain. We were saying (paraphrased for parody):

> ``#if defined(THING)`` was good enough for my ancestors, it's good enough for me!

all while shaking our canes from our Committee rocking chairs. It's easy to get stuck in this. We have to be simple and **fundamental**. We have to be **safe**. We can't just go adding things willy-nilly to the Standard. So what if it's "helpful", or "teachable", or "consistent", it's not a great **value add**. We get curmudgeonly. We get _conservative_. We oppose change, because change itself can introduce risk and responsibility, and we especially oppose small changes because it is "not worth it". We don't want to make C compiler implementations complicated. Ease of use, of implementation, of understanding, all that jazz.

But those e-mails -- that connection to you, the developer -- centered us. It gave those of us on the Committee fighting for the paper side the strength to persist. And being able to share your tangible and direct ask for this feature with the Committee in real-time turned around what would have likely ended up as a razor-thin rejection into a pass. Can you imagine?

A small, kind and purposeful effort — some encouraging outreach! — softened the trigraph-engraved, preprocessor-encrusted blocks we call our hearts?

Now, the [ISO process is extremely frustrating](https://twitter.com/isostandards/status/1367138676162105344) and many of us have railed against it (myself included). But. These proposals are written, as either part of work or out of sweaty, volunteered grit and determination, by human beings. The people in the Committee are human beings. We do care. We are not detached from you, or reality.

We try our best to be careful and pragmatic and consider the literal giga-verse of all compilers and implementers and architectures and hardware configurations and compilation targets. We run surveys and poke at compiler implementations and check code and download hundreds of gigabytes of source to clang analyze and `ripgrep` and we ping companies for their feedback and more. We coordinate with other standards, tackle provenance, try to make more sense of Decimal and Binary floating point, leave room for implementations, and more. We also love the tools we use and the perspective we have grown into from taking care of those compilers and static analyzers and sanitizers, because they are our beautiful little darlings who deserve a seat at the table too.

It's a lot of details to sift through and a lot of details to get right, and it's easy to become reflexively conservative when you're not just asking for _that one feature_ from the outside, but sit in the central nexus of every single potential feature that can be added and every single potential tweak.

We **do** get lost in the vastness and inertia!

But advocacy from developers – from our constituents, e-mails no longer than 4 short sentences and sent sincerely + enthusiastically — tipped the scales away from that cold, crushing conservatism that plagues any long-lived Thing.




# Towards a Greater Good

To those who messaged me. And, specifically, to the Y.G. Senior Embedded Developer who sent those words of encouragement to Melanie Blower about the paper; thank you for reminding us of even the little things we can do to make it better for you. There is No Us, Without You,

And You're Great.

Even when you're just in tiny cute e-mail 📧 form. 💚

{% include anchors.html %}
