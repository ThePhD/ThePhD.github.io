---
layout: post
title: Generating Motivation
permalink: /generating-motivation
redirect_from:
  - /2018/10/22/Generating-Motivation.html
feature-img: "assets/img/2018-10-22/stay-focused.jpg"
thumbnail: "assets/img/2018-10-22/flowchart-point.jpg"
tags: [C++, sol, sol2, support, motivation, ⌨️]
excerpt_separator: <!--more-->
---


It's time I started thinking seriously about the sustainability and support for all the work I put into C++ and sol2. <!--more--> There are two things I need to address in order to keep fundamentally motivated for sol2, sol3, and C++ in general.



# When ~~Love~~ Excitement Is Not Enough

One of the primary motivations for starting sol2 and supporting it for around 5 years and developing it into the highly competitive and immensely scalable binding layer it is today was that I was learning things. New techniques, interesting language corners from users shoving things into functions I never thought people would, and how to keep a library beautiful while keeping it fast. Well, not just fast: the **fastest** in the world! That is no small feat when it comes to _keeping_ usability the same while constantly changing the internals.

Unfortunately, I do not have a wellspring of excitement in my soul.


## Uh Oh, is that...Boredom?!

The things anyone dreads to feel in a personal project: disinterest and boredom. Tag dispatching over the same set of traits for the 10th time in yet another utility function, the novelty gets lost on me. The once shiny and beautiful C++ is now just the mundane boilerplate required to communicate something that has been done quite a few times before. I am not finding some new corner of C++ or some fantastic technique that really saves me. In fact, more often than not I am running into some limitation of C++ that inhibits my ability to express the things I would like to express. I have wrote papers solving these problems, but they will be in C++23 in the best case.

sol2 (and the upcoming sol3) are reaching a point for me where I'm not actually learning anything from implementing things in the project. Before, there was a serious burn and desire in my soul to keep hacking away and implementing things in sol because it just had such a strong satisfaction kickback for every feature: I learned tons of things, and it was always fresh. Challenges were always new, and supporting people and their desires meant tackling interesting API design, performance, and scalability questions. This new iteration of sol -- sol3 -- will address many long-held concerns around compilation time, and ultimately drastically increase the intuitiveness and usability of the `usertype` API.

But my excitement is still waning.

I spent a good while talking to other people and looking at how they maintain their open source projects. I do not want to go the route of just abandoning sol2 to be a ghost repository, which seemed to be quite a common theme. Being bored is an absolute killer when there is an opensource project on the line, because it is strictly a time sink for little return-on-investment, especially if it is freely available with no real requirements in return (e.g., MIT Licensed). I don't want to get bored of sol2, but I can sort of feel it creeping into my soul. I am still making solid progress on sol3 and have finished completely rewriting the `usertype` system, but now I have quite a bit of extra work to do from new feature requests and test re-engineering and goodness does it hit me hard every time I open up the files those 2K assertions...!



# Generating Motivation

What I realized I need was a positive feedback loop. This previously came in the form of overcoming challenges in my own understanding and supreme glee in seeing what was just a personal project that I had no actual stake or software running with blossom into something usable by the general public. Now, I need to recreate that positive feedback effect or risk the urge of running myself aground with a lack of motivation and suffering the same fate as Selene or LuaBridge or others. So...


## Get Motivated: Rewarding Myself

This came from looking at Jonathan Müller (also known as foonathan)'s blog and work. He does something incredibly interesting, where he sets up a Patreon and for every library, blog post, and other thing he accomplished not directly related to his schooling or work, he bundles them up together and makes a "Productive Week" summary out of it. He engages his supporters on Patreon, giving them exclusive content into the inner workings and machinations of his thoughts and ideas.

From my perspective, as a person who participates on the content-receiver end of it, it is a very worthwhile and effective loop! It probably helps him feel a lot more engaged in his work and gives him a sense of responsibility for the quality of the things he ships. It also gives him a constant reminder to turn around and share, which is something that I feel is integral to my ability to keep on making useful things for general public consumption. I would like to feel that kind of spark again, so I have set up my own Patreon!

[![It's alive!](/assets/img/2018-10-22/patreon-splash.jpg)](https://www.patreon.com/thephd)

There's already a first post in there (for patrons only). I hope to keep pushing out weekly and monthly content. Part of staying motivated is engaging with the people that help support me and help make sol3 great, whether they're volunteering time to contribute to the project or money to make sure the project can stay afloat even as I continue to pursue my dream of finishing a Bachelor's degree, getting a Master's degree, and finally obtaining my nick-namesake in a Doctorate.


## Get Motivated: Doing New Things

Doing new things (or even things slightly different) is a good way to keep a good perspective and not get lost in a few trees. Seeing the whole forest is good, and part of that was coming up from my own needs in sol2/3 and looking to the bigger, brighter picture.

Writing papers for C++ Standardization and implementing them, thankfully, is always an invigorating and fun experience. I do not think I will tire of it, really! One thing I was thinking about doing is that maybe I should start championing proposals that I do not start or fully own. For example, [p0330 - Literal Suffixes for ptrdiff_t and size_t](/_vendor/future_cxx/papers/d0330.html) is not a paper I started, but I picked it up, helped clean it up a bit, rewrote it, and now I am both its primary author and champion. I am also going to be picking up the `std::overload` paper as well as championing quite a few of my own.

This has been a very satisfying experience. There are a number of papers that people propose once that are genuinely good ideas, but do not have the time or energy to push them through the C++ Committee. I think I am getting a very good grasp of that, and so would potentially like to offer that service to others, maybe even to patrons of the Patreon! It's still an idea I am toying with and not something I can act on right away: I have school to think about and my final year of classes coming up so I can get my Bachelor's and start looking at Master's/industry! But... it's an idea in my back pocket.

That is all for now. I hope you will check out the Patreon if it interests you, and perhaps help me engage in keeping sol supported and my motivation in a good place!

Until next time! ♥
