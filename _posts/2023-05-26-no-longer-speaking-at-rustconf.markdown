---
layout: post
title: "I Am No Longer Speaking at RustConf 2023"
permalink: /i-am-no-longer-speaking-at-rustconf-2023
feature-img: "/assets/img/cake/cakethulhu - memedrawover - banner.png"
thumbnail: "/assets/img/cake/cakethulhu - memedrawover - banner.png"
tags: [Rust, RustConf, Professionalism]
excerpt_separator: <!--more-->
draft: true
---

I will no longer be speaking at [RustConf 2023](https://web.archive.org/web/20230526194949/https://rustconf.com/speakers/bjo-rkus-dorkus) about [A (Possible) Future for Compile-Time Programming](https://web.archive.org/web/20230526195015/https://rustconf.com/schedule/a-very-quick-walkthrough-of-possible-compile-time-reflection-).<!--more-->


# What?

The decision happened after I was reached out to by RustConf 2023 organizers in a call on May 26, 2023 around 11:45 AM US Pacific Time. The call was about taking my talk and degrading it from a "Keynote" to a "Regular Talk". I figured this was for scheduling or timeslot reasons or because they found someone who was a better fit, but none of these were the case.



# Why?

There are 2 "why"s to answer here.

The first is why I am writing this blog post. I noticed that the reason I got an influx of follows is because the Rust Conference program was released, and it has my twitter on it. Sure enough a bunch of people (especially Rust-related and ones with the typical calling of the programming language, the sacred "🦀"!) followed my socials. I was also told congratulations by a few close friends who saw it and were paying attention (again, thank you!). If I have to individually explain the reasoning and behavior that is going on here to each person, my fingers will quite literally fall off and I absolutely do not want to waste everyone's time by giving partial retellings over and over and over and over again.

The second reason is a lot weirder than I had anticipated or thought about, and has to do with *how* this happened and the strange behavior coming from the Rust Project about this. But first, a little background.




# How?

I did not submit a talk to RustConf 2023. I had to be given a special link to register my talk in Sessionize *after* the deadline because I was not anticipating to do a talk until 2024, when the Compile-Time Reflection work would be done.

The reason I had to submit the talk specifically was because it was at the behest of the conference organizers. More specifically, I was nominated by "Rust Project Leadership" (to be exact with the wording) to give a keynote (start of the day, shared slot with somebody else, 30 minutes) about something Rust-related. Given that my only public Rust work up until then was my [Mega Midterm-Report on Compile-Time Reflection in Conjunction with one other group](https://soasis.org/posts/a-mirror-for-rust-a-plan-for-generic-compile-time-introspection-in-rust/), I figured that was the best thing I could give a talk on.

I was not quiet or shy about this; I was very vocal with everyone I was in contact — both inside RustConf leadership and with Rust Project leadership who were even remotely in the same Discords, etc. as me — about what I would give a talk on. I would talk about compile-time programming, my ideas for it sourced from the Rust Foundation Grant Work I was helping with, and make sure it was very clear that it was a **possible** future for Compile-Time Programming, not the exact future. The title of my talk is also very clear:

> A (Very Quick) Walkthrough of **Possible** Compile-Time Reflection?!

Emphasis mine on the word "Possible". I also feel like I have been more than forthcoming that this is nobody's ideas for compile-time programming but our own. That is, here is the top-level disclaimer at the very beginning of the Compile-Time report:

> As a general-purpose disclaimer, while we have spoken with a large number of individuals in specific places about how to achieve what we are going to write about and report on here, it should be expressly noted that no individual mentioned here has any recorded support position for this work. Whatever assertions are contained below, it is critical to recognize that we have one and only one goal in mind for this project and that goal was developed solely under our — Shepherd Oasis’s — own ideals and efforts. Our opinions are our own and do not reflect any other entity, individual, project and/or organization mentioned in this report.
>
> — [A Mirror for Rust: Compile-Time Reflection Report, §Primary Motivation, Paragraph 2](https://soasis.org/posts/a-mirror-for-rust-a-plan-for-generic-compile-time-introspection-in-rust/#as-a-general-purpose-disclaimer-while-we-have-spoken-with-a-larg)

I cannot imagine it is not clear whose ideas these were and that they did not represent Rust Foundation, Rust Project, or anyone else's orthodoxy but that this was a technical report based exclusively on our ability to go through the syntax, data, and benefits of existing code. You can imagine my surprise, then, when I was told today by the Rust Conference organizer that my talk "did not want to be endorsed by the Rust Project, and that is what a keynote is meant to be for".




# ... Huh?

Yeah. To be clear, RustConf's keynotes often cover topics both near and far to the Rust Project's goals and needs. Never before has the keynote suddenly had an almost holy indication of the direction of the Rust Project's goals, nor did presenting about any given topic before seem to have the explicit goal of endorsing a specific result or a specific future. All of their talks are available through the RustConf 2022, 2021, etc. playlists from the [@RustVideos channel](https://www.youtube.com/@RustVideos); verifying this is as easy as going back through the talks to note that there is plenty of "heterodox" and competing viewpoints spoken of by past RustConf keynoters and invited speakers. It is also explicit policy of RustConf in prior years (2022 and earlier) to bring in speakers with different / outside viewpoints, so even **if** this keynote's contents were not endorsed by the Rust Project explicitly, that only increases its appeal to the at least intended and stated values.

This is, again, also in the face of: 

- a conference talk title talking about the still-developing nature of this work;
- a clear indication in publicly-available materials that the work is not endorsed by any individual, organization, and/or collective; and,
- a clear indication from me, personally, to the RustConf Organizers and Rust Project leadership that this is what I would be presenting on and its contents before I committed to anything.

At no point should anyone have thought that I would be capable of misrepresenting what the Rust Project wants out of its ecosystems, or that I had somehow overtaken competing proposals in the space. As I always do with technical work that I am involved in, it is made to be entirely transparent, deeply open, and very clear based on the technical merits of the work. All of the explanations in the Midterm Compile-Time Programming Report for Rust were technical in nature, and critiqued all of the available work based on the merits of its work. Therefore, it is baffling to first be nominated and accepted as a speaker, rush to ensure I provide the Conference Organizers with everything they need to present the materials accurately to the public with the **explicit blessing** of the Rust Project Leadership and at its direct behest, only for it to be pulled back later.

It is also deeply confusing and ultimately insulting for them not to contact me beforehand and simply ask me if I would disclaimer my work to make it clear that they did not explicitly endorse this direction. Multiple times before the RustConf schedule and program was released, I made it obscenely clear that there was not going going to be an RFC for the work I was talking about ("Pre-RFC" is the exact wording I used when talking to individuals involved with Rust Project Leadership), that this might bias folks, and whether or not it would be **okay to do this**. Individuals in contact with me both inside and outside RustConf leadership made it abundantly clearly that this topic was perfectly fine. Furthermore, they had already met to discuss my work **before** hand, so at no point should anyone be confused about what my intentions and goals are.




# Spending My Resources to Play Weird Games

It is patently obvious that somebody is pulling very weird strings behind the scenes here. At multiple junctures I said "it would be fine if we did not do this" or "I am unsure why they picked me", but at multiple point I was told I should *absolutely keep going* and it was **explicitly** encouraged for me to go all the way with a keynote! For Rust Project-involved individuals to speak to me, directly and without room for misinterpretation, that I should be doing this after **their** group pushed me to the forefront is bizarre.

It is also doubly dubious that rather than any individual contacting me directly from Rust Project leadership — including the folks who nominated me in the first place — I was given this mandate from RustConf Organizers who admitted they had no influence over this decision. If someone objects to the direction and content of the Midterm Compile-Time programming report, I have been and will continue to be available to discuss the merit of my work and whether or not it is a good fit for the direction of Rust in general. But, to date:

- Shepherd's Oasis and I have not received any e-mail communication regarding the work or any concerns from the Rust Project;
- Shepherd's Oasis and I have not been given any warnings or asked to reconsider things through any kind of instant messaging communication or otherwise (Zulip, Discord, etc.) from the Rust Project; and,
- Shepherd's Oasis and I were not the original ones seeking to give presentations about this subject material until we were **explicitly asked to**.

The sudden reversal smacks of shadowy decisions that are non-transparent to normal contributors like myself. It is a brutal introduction to the way the Rust Project **actually** does business that is not covered by its publicly-available Procedures and Practices and absolutely not at all mentioned in its Code of Conduct.

As the Rust Foundation had trouble with its trademark rollout and [the Rust Project presented itself](https://twitter.com/m_ou_se/status/1537836681839116290) as the [capable group that can do the right thing](https://twitter.com/m_ou_se/status/1645720730955382785), I find myself in the opposite situation here. The Rust Foundation has handled the grant work with utmost grace, respect, and professionalism for myself and Shepherd's time. Contrarily, the Rust Project deigned to effectively pass several mandates down through an opaque process that affected me, while refusing to air to-this-minute unknown grievances with the direction of the Compile-Time Midterm Report. I have already committed a lot of time to continue exploring the direction detailed in the report; the entire goal of that report was to let everyone know **early** what we were doing and where we wanted to go. We even held off on writing an RFC for the things we wanted because we did not want to pitch a direction that the Rust Project was uncomfortable with! As of yet nobody has given an even cursory rebuttal to the report or parts of its contents other than users and a few Rust contributors spending a good amount of time either

- yearning for a different direction entirely (á la Zig's `comptime` direction), which I responded to directly about its implications and impact and challenges; or,
- that it was "incompatible" with the current Rust model on CTFE (how and why was never fully detailed, but most of these conversations have not gone into sufficient depth to get to that point or explain what specific parts of the compiler or technology would make this impossible to implement).

To invite me as a keynote speaker and then quietly cut my legs at the last minute?

![An anthropomorphic mouse staring at a cell phone with their eyes wide and their pupils a little beady. Their cheeks are a little pulled in from taking a long drag on a blunt, concern all over their face from the thing they are viewing on their phone.](/assets/img/cake/cakethulhu%20-%20memedrawover.png)

Absolutely not. This is a mark of both vindictive behavior and severe unprofessionalism that I expected from various organizations I am forced to interact with from day to day as a human being living in a flawed world, but not the Rust Project. Sure, I need to ask the organization that was going to cover expenses to refund everything before its too late (which, perhaps as the only benefit to this situation, they have revealed themselves early enough that this should be easy to do). And sure, I no longer get to stand up at RustConf — the most prestigious Rust-affiliated conference there is — and talk. (Originally, I was thinking of attending the London Rust Conference to talk about this instead, sometime in 2024 if it was still around.) This was after me repeatedly trying to weasel my way out of the engagement but failing to tame my usual too-powerful urge to talk about X or Y.

But, truly?




# There is only One Winning Move.

If this is a game, then as painful as it is and as much as I love indulging my neurodivergence to get up in front of a large body of individuals and start infodumping my heart out,

I must not play.

If I give the talk after it has been downgraded, then I acquiesce that this grotesque behavior is something I am personally/professionally okay with.

If I try to negotiate my way back towards a keynote while constantly asking for reasoning for this out-of-the-blue decision, then I am accepting their framing that their decision making and process is valid and that I must abide by their opaque governance.

I choose to do neither. 💚

(Art by [Cake](https://cakethulhu.carrd.co/), go follow them everywhere and enjoy their adorable, wonderful art!)
