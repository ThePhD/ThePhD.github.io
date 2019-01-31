---
layout: post
title: C++ Now 2018 Trip Report
permalink: /c++now-2018-trip-report
redirect_from: 
  - /2018/05/15/C++-Now-Trip-Report.html
  - /2018/05/15/C++Now-Trip-Report.html
feature-img: "assets/img/2018-05-15/C++Now feature.jpg"
thumbnail: "assets/img/2018-05-15/C++Now feature.jpg"
tags: [C++, C++ Now, speaking, conferences, 🤝, 📣, 📜]
excerpt_separator: <!--more-->
---

The trip for the [C++ Now 2018](http://cppnow.org/) conference in Aspen, Colorado has ended! I gave 2 lightning talks and my talk about sol2, and I think it went pretty well!

<!--more-->

Aspen, Colorado is an absolutely gorgeous area. When I first got there, it felt like I was taking a vacation! Of course, a vacation might have had me getting a little more sleep than I ended up catching that week, but every night staying up until 12 or 1 AM in the morning talking about C++, techniques, experiences and ideas was entirely well worth it. It is rare I get to geek out in person... and I got to do a whole **week** of it in beautiful Aspen!

![Breathtaking!]({{ site.baseurl }}/assets/img/2018-05-15/C++Now Scenery.jpg)

Volunteering for a conference was a lot of fun! You had to attend a talk at every single track, help move food and clean up, and manage a few conference tasks like registration and guide people at the beginning of the conference. All in all much of it was done ahead of time and I was just gluing together some of the pieces other people had prepared to make things easier. We also got to test run the new website-based, schedule-based digital timers for C++Now 2018, which in my opinion were an immense success!

## Listening: initializer_lists and Unicode and more!

My favorite talk was Jason Turner ([@lefticus](https://twitter.com/lefticus))'s "Initializer Lists are Broken, Let's Fix Them", albeit I hesitate to term it your usual lecture-style talk! It was more of a guided, interactive performance exploration of `initializer_list`, what it means, and ways to improve the design. Jason has some *pretty fantastic* slide tech, too, making Compiler Explorer pop up at a single click on his slides! It was really engaging and demonstrated some practical implications for API designers who were looking for both speed and API design tradeoffs. I am more or less intensely interested in exactly this design space, so the talk was immensely practical for me!

Another great two-part talk was Zach Laine's about [Boost.Text](https://github.com/tzlaine/text), which was both a great overview and a wonderful deep-dive into designing a practical API that can handle Unicode. Seeing the design choices he made and tackled in getting a full collation-handling, grapheme-identifying, normalization-using text processing library was very interesting, plus all the interesting things about human languages that come with trying to make it all work out properly in code. The world is an amazingly complex place, and Text is just one area where you see all the complexities and not-so-clean but entirely rational ways in which human beings communicate with one another. I hope get some of his work in hand and start benchmarking it against other solutions in the same space.

Bob Steagall's talk in the same space also went through these subjects, but from the perspective of performing blazing fast utf8 decoding! You can read some of his experience and about his talk at [Bob's Trip Report](https://bobsteagall.com/2018/05/13/cppnow-2018-trip-report/).

## Speaking: sol2, Lightning, and More!

On Thursday, I gave my talk about [sol2](http://sol2.rtfd.io/). The slides are [here](/presentations/sol2/C++ Now/2018/2018.05.10 - ThePhD - Compile Fast, Run Faster, Scale Forever.pdf), which in and of themselves I think turned out very nicely. It's a bit unfortunate that usertypes were relegated to a demo but there's more than enough documentation and examples in sol2 for someone to steep themselves into it if they really liked it.

I think my talk was received well. Someone in the audience actually **used** sol2 as well, which really made it hit home that I was developing a library that was supposed to be usable for other people. It really had reinvigorated me to have that level of responsibility and ownership over some code, and that my care of it affects the performance and ability for many, many applications.

Plus, it's really awesome when someone as amazing as [Odin Holmes](https://twitter.com/odinthenerd/status/995495656054710272) tells me that I did a good job and that he likes my talk!

I did not get a chance to publish my updated benchmarks at my talk, so as a follow-up I'll be placing my updated benchmarking numbers in sol2's documentation and in a new blog post soon!

I gave 2 additional lightning talks while I was there too. One was about [`std::embed`](https://rawgit.com/ThePhD/embed/master/papers/P1040%20-%20embed.html), a way to lift a resource into a compile-time view. I gave it as a bit of a cheesy limerick, and then made comments about it afterwards! Most people seem to really like the idea, but we will see how it goes in the C++ Standards Committee.

The other was a less technical and more personal talk about the idea of where I belong.

## "We Are Glad You Are Here"

There are a lot of criteria people have for feeling like they belong. It usually boils down to feeling like there's a community that you can tap into and share with and be a productive member of. It also comes from not feeling like you are being pushed out for reasons which are either beyond your control or have nothing to do with the core of your character. I never really felt like I had a community I could tap into, for various reasons. I talked a little bit about that, and the response was overwhelming.

From a nerd who grew up in a Hippie Commune to the son of an immigrating Polish family who spoke very little English, the story seemed to resonate with many people. I was a bit sad that so many people felt alone, but extremely encouraged that my experience was not just a "People of Color" feeling or a singular thing. There is a shared experience when you are smart and bright in a way that is not expected of you. And the overwhelming end goal is simple: it is going to be okay, so long as you do not give up that thing that helps shape who you are for the sake of fitting an expectation other people have of you.

I am elated that C++Now was the place I ended up from pursuing my unique interests, drive and goals. It was my first C++ conference and I felt incredibly welcome. My skin color was not an issue: I was judged as a human being, student, and upon the merits of my work alone.

These are the kinds of places I live for; the preconceived notions and ideas of what I might be capable of because what I look like were left at the door, and I was viewed as the individual I signed up to be. I am **first** a Student, a Volunteer, a presenter and a software developer who is immensely interested in learning, growing and ultimately sharing what I had learned, accomplished, and experienced. I never had to think that I was being judged first for being dark-skinned above all else, and that was liberating.

I cannot wait to go to another C++ conference and dive into that kind of atmosphere again! But for now, my goal is to go to the next C++ Standards Committee meeting and champion my `std::embed` paper.

Until next time! ♥

<sup><sup><sup><sup>Const West is Const Best!</sup></sup></sup></sup>
