---
layout: post
title: presentations
feature-img: "assets/img/portfolio/presentations.png"
img: "assets/img/portfolio/presentations.png"
date: December 21st, 2019
tags: [C++, speaking, conferences, python, Portfolio, 🚌, ⌨️, 🤝, 📣, 🎧]
---

I have given presentations and spoken on several occasions! Here are the presentations I have done so far and some information about them.



# Video Presentations

## CppCon 2019 - Invited Speaker

<div style="text-align:center"><iframe width="560" height="315" src="https://youtu.be/FQHofyOgQtM" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe></div>

[Slides](/presentations/unicode/Meeting C++/2019/2019.11.16 - Catching ⬆️ - Unicode for C++ in Greater Detail - ThePhD - Meeting C++.pdf). This is the next talk in the series of talks on Unicode. Some of the uses of the benefits of error handling are explained here, progress with the C Standard, as well as some other tenants of the library. This is an ongoing talk series and part of a larger [project on text](/portfolio/text).


## CppCon 2019 - Invited Speaker

<div style="text-align:center"><iframe width="560" height="315" src="https://www.youtube-nocookie.com/embed/BdUipluIf1E" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe></div>

[Slides](/presentations/unicode/CppCon/2019/2019.09.20 - Catching ⬆️ - The (Baseline) Unicode Plan for C++23 - ThePhD - CppCon 2019.pdf). The beginning of a series of talks on Text and how we should fix it for C++. This talk dives into why text inside of any of C++'s current components is a dead end task, the non-portability of `char` and `wchar_t`, as well as the problems with tackling encoding today in the Standard Library. It then proposes an overall framework for how to handle text encoding in C++ in the future. This is an ongoing talk series and part of a larger [project on text](/portfolio/text).


## C++Now 2019 - Invited Speaker

<div style="text-align:center"><iframe width="560" height="315" src="https://www.youtube-nocookie.com/embed/aZNhSOIvv1Q" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe></div>

[Slides](/presentations/sol2/C++ Now/2019/The Plan for Tomorrow - Compile-Time Extension Points in C++.pdf). This talk came from a challenge by Eric Fiselier to research extension points in C++ for P1132 - std::out_ptr. I went further and decided to survey the entire landscape of C++ and its extension points, how we currently handle those extension points, and how we can, should and might handle them in the future. This covers the entire landscape of applicable compile-time extension points for libraries in C++. Runtime extension points (hot reloading, DLL/function hooking, inserting code in left-behind "bubbles" of space, RPath, and more) is still a talk yet to be given (perhaps, by you?!).


## CppCon 2018 - Invited Speaker

<div style="text-align:center"><iframe width="560" height="315" src="https://www.youtube-nocookie.com/embed/xQAmGBfKnas" frameborder="0" allow="autoplay; encrypted-media" allowfullscreen></iframe></div>

[Slides](/presentations/sol2/CppCon/2018/2018.09.28 - ThePhD - Scripting at the Speed of Thought.pdf). This is the (so far) last talk I have given about sol2 and its now-released additional version, sol3. It talks about the changes made to accommodate new ideas, performance improvements computed the very same day of the presentation, and even more! It was exciting to speak about sol3's future, and actually deliver on it. There's a lot more to do with sol3, but for now I am very happy where it ends up and this talk explores a lot of why I am happy with it.


## C++Now 2018 - Invited Speaker

<div style="text-align:center"><iframe width="560" height="315" src="https://www.youtube-nocookie.com/embed/0Lwy4_sKeJM" frameborder="0" allow="autoplay; encrypted-media" allowfullscreen></iframe></div>

[Slides](/presentations/sol2/C++ Now/2018/2018.05.10 - ThePhD - Compile Fast, Run Faster, Scale Forever.pdf). This is one of my favorite presentations of sol2. I felt I was really well-prepared and animated for one of my first big conference talks in front of a large group of individuals since I presented about _Bacillus Anthracis_ in 2010 for the ABRCMS. I also met a ton of great people at this conference!


## C++Now 2018 - Lightning Talk I

<div style="text-align:center"><iframe width="560" height="315" src="https://www.youtube-nocookie.com/embed/EG5v7CSmO3s" frameborder="0" allow="autoplay; encrypted-media" allowfullscreen></iframe></div>

[Slides](/presentations/standards/C++ Now/2018/std.embed - a poem.pdf). This was a short little poem about the upcoming [std::embed proposal](vendor/future_cxx/papers/d1040.html). Was fun to do, delivery was decent!


## C++Now 2018 - Lightning Talk II

<div style="text-align:center"><iframe width="560" height="315" src="https://www.youtube-nocookie.com/embed/Vl8OK1hDYUg" frameborder="0" allow="autoplay; encrypted-media" allowfullscreen></iframe></div>

[Slides](/presentations/personal/C++ Now/2018/We Are Glad You Are Here.pdf). This is a more personal lightning talk about what it means to be part of a community and how I didn't really have one, growing up. That changed when I got into programming.


## Lua Workshop 2016

<div style="text-align:center"><iframe width="560" height="315" src="https://www.youtube-nocookie.com/embed/NAox5UsjbUM" frameborder="0" allow="autoplay; encrypted-media" allowfullscreen></iframe></div>

[Slides](/presentations/sol2/Lua Workshop/2016/2016.10.14 - ThePhD - No Overhead C Abstraction.pdf). Presentation for Lua Workshop 2016 about the early version of sol2. Includes some of the core of wrapping the Lua C API and the details of the usertype API at the time!



# Slide Decks

Below are some slide decks for various presentations I've given that were not recorded. Click the image to get to the slides!


## Study Group 16: Unicode - March 2018

[![A Rudimentary Unicode Abstraction - Attempting to wrangle unicode and more](/assets/img/thumbnails/rudimentary-unicode.png)](/presentations/unicode/sg16/2018.03.07 - ThePhD - a rudimentary unicode abstraction.pdf)

A presentation done for SG16 - Unicode when I first joined the group and some of the design decisions I made. Covers encoding and having an abstraction that sits on top of other containers, and the evolution of how I got there.


## Boston C++ Meetup - February 2018

[![Biting the CMake Bullet](/assets/img/thumbnails/cmake-bullet.png)](/presentations/CMake/Boston C++ Meetup/2018.02.06 - ThePhD - Biting the CMake Bullet.pdf)

Someone asked about CMake during the January 2018 meetup, so I threw together a presentation of what I knew about CMake. It is by no means an expert presentation, but I made one nonetheless for a 45 minute presentation as a non-expert. I think talking from a perspective of a learner gives valuable insight into things!


## Boston C++ Meetup - November 2017

[![Wrapping Lua C in C++ - Efficiently and with a Touch of Magic](/assets/img/thumbnails/wrapping-lua.png)](/presentations/sol2/Boston C++ Meetup/2017.11.08 - ThePhD - Wrapping Lua C in C++.pdf)

An early presentation of sol2 at my C++ Meetup. I did not think it would be so much fun, but it was! I went a tad overtime, but that was because I was so zealous about the topic, and quite a few people were very interested in the actual contents of the conversation! It was quite nice. It was about a 50 minute presentation or so.
