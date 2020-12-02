---
layout: post
title: The (Late) CppCon 2018 Trip Report
permalink: /cppcon-2018-trip-report
feature-img: "assets/img/2018-11-26/CppCon Meydenbauer.jpg"
thumbnail: "assets/img/2018-11-26/CppCon Logo.png"
tags: [C++, CppCon, speaking, conferences, 🤝, 📣, 📜]
excerpt_separator: <!--more-->
---


"Late", aha. This is becoming a habit, isn't it? In an earlier post, I talked about the [C++ Rapperswil Committee meeting]({% post_url 2018-08-21-The Late Rapperswil Trip Report%}) and said I'd be at CppCon 2018 volunteering and doing work! <!--more-->  And I did just that.

This conference was quite different from C++Now. It was much larger, much more intimidating, and I actually met Scott Meyers accidentally while coming to the Meydenbauer on Sunday Night to see if I could open some rooms ahead of time and get a feel for how they would be.

To my utter embarrassment, I did not realize it was Scott Meyers. I had tried all the doors on the first floor, then the third floor, then finally all the fourth floor doors. I was coming back from trying all of them when I saw someone jiggling the handles like me. I figured I would save him some time: "They're all locked." And from there we sort of just started talking. He said he was giving a class, and I was like "Oh wow really? Which class? Oh, and I didn't catch your name."

"Oh, I'm Scott Meyers."

![... Wot.](/assets/img/2018-11-26/surprised-cat.jpg "u wot, m8?")

I maybe had a little fanboy mental short circuit. I even misspelled the location like a champ when I tweeted about it. Thankfully, this was a good run-in! I even got to talk a little bit about my future with him and he even shared some of how his life went, which definitely gave me quite the perspective.

It wasn't just Scott Meyers, though. I got to see even more C++ legends, and I handled it a lot better because I got the super fanboy out of my system! I met Kate Gregory, Patricia Aas, Nicole Mazzuca, Isabella Muerte, Simon Brand, Andrei Alexandrescu and **quite** a few other really amazing folks. There were also individuals who only after meeting them did I realize the full weight of how lucky I was to shake their hand: from Rob Maynard (of CMake fame) to Rob Irving (of CppCast!) to Hubert Tong (who I saw at Core in Rapperswil!) and countless others, I couldn't believe the sheer concentration of individuals there who had this vast pool of C++ (and C++-related) knowledge.

I also got to meet some other really cool people that weren't "legends", but honestly were well on their way there! One of the poster presenters Peter Goodman from Trail of Bits. He talked to me in detail over the pre-Registration dinner, where I learned all about the work he did and the Remill library. It's actually open source: [so check it out yourself](https://github.com/trailofbits/remill)! It's not my area of expertise but it was fascinating to learn about it from Peter.

I also got to meet Todor, who was part of VMWare's team. We didn't get to speak much to each other after registration night, unfortunately, despite working together on the [std::out_ptr](/_vendor/future_cxx/papers/d1132.html) paper. This is actually a bit of a cautionary tale: you can come to CppCon and if you spend a lot of time just trying to meet every single person, you will not be able to without extreme dedication! I did not get a chance to talk to Todor again, and I missed talking with Michael Caisse after the very first night of registration too...!


# Right, The Talks

There were a lot of talks. Way too many for me to digest in one meeting: I tried to prioritize seeing things a bit more relevant to my direction interests! I also had seen versions of the talks before so I did not have to see everything.

One of my 2 favorite talks was already mentioned in [the optional pregame](/sandiego-2018-pregame-optional), but I'll mention it here again: 

<iframe width="700" height="315" src="https://www.youtube-nocookie.com/embed/J4A2B9eexiw" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

This talk was very thorough in going through everything that goes into making a wrapper type. It's incredibly important to understand when you're taking things and you're trying to "emulate" the behavior as best as possible within a given set of constraints. Simon's talks are also incredibly informative and nice to listen to, so of course I ended up going to 3 out of 4 of the talks he had! But, I also went to some other talks too, particularly one from the famous Arthur O'Dwyer:

<iframe width="700" height="315" src="https://www.youtube-nocookie.com/embed/IejdKidUwIg" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

He's done a lot of work with allocators and memory, and in recent days has been working on something called `TriviallyRelocatable`. I only caught part of this talk in C++Now 2018 and watched the video, so I wanted to get the fullness of the talk at CppCon 2018. The very first slides were helpful to me, because I was struggling to reconcile what a value was versus what an object was and how it related to things like optional references and how to wrap things.

Either way, defining an allocator as a handle to a heap makes a lot of sense. It also separates the two out: the total heap storage backing the allocator versus the allocator interface itself. With this kind of separation its easy to be able to reason about allocators, especially in the case of polymorphic memory resource (`pmr`) allocators which take this talk title quite literally in its usage. It's a sensible separation that allows the concept to scale quite far, even in the face of POCMA (Propagate On Container Move Assignment).

I still don't know full well about the whole hierarchy, but seeing Arthur's talk and listening to him detail the various bits about how one can store and handle state in a `memory_resource` and handle it with an allocator has proven to be very useful to some code I've been messing around with! And speaking of things useful to code I want to write...

<iframe width="560" height="315" src="https://www.youtube-nocookie.com/embed/GC4cp4U2f2E" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

This talk was so much fun! If you're already embroiled in error handling patterns like I am, you won't really learn too much, but near the end they propose some **wicked awesome** syntaxes for a potential C++23 world that has Herbceptions (Deterministic Static Exceptions, [p1095](http://wg21.link/p1095)), the Workflow Operator [p1282](https://wg21.link/p1282), and a good `std::excepted`. `std::expected` was also covered this year from another talk, [Expect the Expected](https://www.youtube.com/watch?v=PH4WBuE1BHI), that was pretty great! But again, not really terribly new information for me since I'd already picked this stuff up earlier from Andrei Alexandrescu's MSDN / Channel 9 talk. It was still entertaining to listen to my first Andrei Alexandrescu talk in person!


<iframe width="700" height="315" src="https://www.youtube.com/embed/IupP8AFrOJk" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

This talk is a general overview of how to handle vulnerabilities. It's not quite similar to Matthew Butler's Secure Coding Practices talk, but just like Mr. Butler's talk it help put a little bit of the fear of ~~God~~ Security in me. A few times I've worked on server code before and had to tangentially be responsible for security after cleaning up the codebase and drastically improving performance, and it's terrifying to have to sit down and try to model all the ways things will go wrong and all the ways you have to be prepared. I even was once in charge of picking the password and hashing scheme for a server, and did I really stall on that feature as I went through every single cryptographic one-way hashing algorithm in recent history, salt and pepper techniques, and other secure practices...

Security is hard. Handling vulnerabilities is a bit easier than the actual security bit I think. Except when they come from attack vectors like the ones that Patricia started talking about at the beginning, then it's almost impossible unless you really box your private data in, avoid storing it unless absolutely necessary, and always err on the side of deletion / scorched earth if you ever have a shadow of a doubt that the data will be solely in your control. Handling responsible reporting and challenging poor company oversight is an important thing to do, too! I hope I have the spine to do it when I get a real job. And despite just being a student at a huge conference like this...


# So Much To Do

So little time! I didn't get to make it to the Boost Dinner (but managed to find Michael Caisse twice, which was great, albeit I didn't get to meet his friends from Ciere). Some days had *multiple* events going on in the same night (e.g. the Boost Dinner clashed with the [#include](https://www.includecpp.org) Dinner)

And to top it off, I had my own talk to give.


# My Talk

Yes, I had one! It was about the future of sol2 named -- you guessed it -- sol3. sol3 is still under active development because boy howdy did I completely underestimate how much work it is to have so many proposals and other things to do with C++.

To be honest, I thought I bombed my talk. And to top that feeling off, it was in the same slot as Odin Holmes, Matt Godbolt, Hyrum Wright, and Sean Parent. _Yes_, all these legends in the same time slot. This probably means that the conference organizers believed in me enough to put me at that time slot and not fail spectacularly. I'm happy to report that the room was definitely not empty!

<iframe width="700" height="315" src="https://www.youtube-nocookie.com/embed/xQAmGBfKnas" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

As a side note: **nice** thumbnail!

At first, I thought I did horrible at the talk. Mostly because there was a demo mid-way and I snagged a compiler error. I was also bleary-eyed tired, too! But, I think people were less judgmental of that failure purely because of the ending where I stated that this was essentially the most real demo ever: benchmarks were computed _the morning of_ and the implementation was tweaked and changed the night before. Which meant I changed my entire presentation from the one I had rehearsed...! It's scary pulling that off, but it ended up paying off. I had originally wanted the video to never see the light of day, but... well, despite being loopy-tired I seemed to handle it decently. Not my greatest presentation, but still pretty good!

This was also when [I decided to put up my Patreon](/generating-motivation) to help pay for supporting sol2 and sol3. It was a very good idea, because communicating directly and being supported directly by enthusiastic individuals has been immensely helpful in keeping my motivation for the project from simmering out. I even managed to backport all the sol3 fixes into sol2!


# My... Surprise Talk?

Well, not really a talk. This ended up being a panel held on Friday. It was not recorded, which made me a bit sad. I had a suit on and everything (it was the same day as my talk so I got to be on the panel after barely sleeping to run benchmarks / re-do my presentation the night before) and the volunteer duties were wearing into me since I also did a few that day too. I wanted to focus purely on the words people spoke while struggling to make sure I was attentive to everything my co-panelists said (Kate Gregory, CB Bailey, Barbara Geller and Patricia Aas were there on the panel with me, which was great!).

Bryce roped me into being on the panel. And by rope I mean he just asked me plainly and I said Yes. No real pressure or anything, albeit it did leave me a day or two to prepare...! Which made me all the more nervous. But, a lot of people said I did a good job on the panel! (I don't know what a Good Job is supposed to be for a panel, other than speaking clearly and enunciating properly.) I was also a bit of a counter-balance: only black person, only student, etc. etc. I got to give some perspectives that other people maybe had not thought about, which is very much the point: exposing people to different thought processes from different individuals at different points in their life helps give a more complete picture. I do wish we had others on the panel! Maybe one or two more.

I learned a lot. When I approach the industry, I'm lucky: I'm mostly talented enough that I can take the preconceived notions, slights of hand, shitty commentary and other things to toss it behind me and move ahead with my life. I am also incredibly lucky that my single Mom raised me to be a whole and complete human being on my own without looking at any of my physical attributes: this makes me largely resilient to people's fairly crappy attitudes on- and off-line.

I feel like Patricia has a ton of knowledge on the subject of how to navigate dangerous spaces, far more than I do. Her responses and experience input to the panel questions were pretty fantastic. Other people at the `#include` dinner also had a lot of knowledge, where they also did a Women in C++ panel. It was also very encouraging. We had a lot of great questions, especially from people who felt like they did not know what to do. I think we were able to prove a good chunk of useful guidance.

For starters, not everyone is as strong or as resilient as the people on the panel or who do take stands.

Anecdotally, lots of people struggle. And it's not just the "minority" that struggles either: people who are "within the normal" still end up on the receiving end of demeaning and humiliating commentary, jokes, and advances. Several people contributed patches through friends or just straight up did not contribute at all to [the Linux Kernel](https://twitter.com/TartanLlama/status/1042306063234674688) because nobody wanted a piece of that place. One thing that scares me the most about joining the technical industry is something I still don't quite have a name for, but for the meantime I am just going to call the Jerk Problem.


# The Jerk Problem?

The Jerk Problem is a very simple one: the larger community _seems_ to value technical excellence over all other values. This means that decency -- and being polite during such things as discussion or evaluation -- is optional. This is how a lot of people conducted themselves: being successful / making money / having influence _somehow_ justifies their ability to be complete blockheads. And while some people just shrugged their shoulders and coin this "Free Speech", well...

Some other things came for the ride while we brushed issues of solo superstar egotistical impoliteness -- to put it mildly -- under the rug.

But you don't have to take my word for it: take Linus's in his personal interview with the BBC:

> Because I may have my reservations about excessive political correctness, but honestly, I absolutely do not want to be seen as being in the same camp as the low-life scum on the internet that think it's OK to be a white nationalist Nazi, and have some truly nasty misogynistic, homophobic or transphobic behaviour. And those people were complaining about too much political correctness too, and in the process just making my public stance look bad.
> 
> And don't get me wrong, please - I'm not making excuses for some of my own rather strong language. But I do claim that it never ever was any of that kind of nastiness. I got upset with bad code, and people who made excuses for it, and used some pretty strong language in the process. Not good behaviour, but not the racist/etc claptrap some people spout. -- [Linus Torvaldes, September 27th, 2018](https://www.bbc.com/news/technology-45664640)

When you make decency and politeness secondary (or tertiary or just not-a-factor at all), you end up with a culture where anything goes. 

Free Speech, right?

So you can be sexist, homophobic, transphobic, racist, and whatever else you want towards people in environments that are _supposed to be technical and bereft of such nonsense in nature_, but as long as "The Work is Getting Done" it doesn't matter. Eventually -- with a touch of [Holy Ascended +10 Rich Rockstar Developer](https://twitter.com/MrAlanCooper/status/1060553914209071106) sparkles -- you end up here. Individualism (Free Speech, full process control, no sharing, no required coordination) of the past -- rewarded by resounding success and piles of cash -- struggles against a team-based ethic (diversity, inclusivity, accessibility, common ground, politeness and good communication) and thusly continue war with one another.

# So What Are You Gonna Do, Derpstorm?

I'm glad you asked! I actually do not intend to partake in the greater war. All I know is that I have to do what I usually do in this situation, which is put on my Token Black Person clothing and go speak at as many conferences as I can to prove that someone with my skin color can be technically excellent. Because -- for some damn reason -- nobody believes I have dark skin ever when they see my videos or meet me. Which results in hilarious reactions:

![Lel.](/assets/img/2018-11-26/Asian.png)

And sometimes, you get hit with this priceless rigmarole:

![Deep Sigh Intensifies.](/assets/img/2018-11-26/rigmarole.jpg)

In fact, some people in my past internships thought I was a woman because I was too jolly in my intra-company chatting and because of my name (avatar wasn't set up yet). (Perhaps I'll tell the story of how their behavior and demeanor towards me changed utterly once they saw my face, but this article is long enough.) Nobody assumes I have dark skin. Because _that's just crazy how could you be black?!_ Because, as we know, there are no Black Engineers! We're a rare breed, or something. During my first internship, this is what I was told: "you are the first black software engineer I have ever seen in my entire life.".

It makes for some pretty great chat and soundbites, sometimes. But the overwhelming impression is that I should not be here to begin with. It's unexpected. And for some people, that's a good surprise (but still a problem). Other people think it's a bad idea I'm here to begin with, which is much more of a problem. Both are still a problem because it should not matter what my skin color is when I do the same technical work, and yet somehow it works its way into conversations and worms its way into my purview whenever people find out what I look like.

And that's not okay.

But! That's just how it'll be for the next 20 years, until some of you work as hard as you can to get up here. Even though I actually am not fond of speaking at conferences or giving presentations (honestly, I could just be at home, programming, instead of preparing and giving these talks) I must keep doing so. I need to keep showing people that not only am I beyond competent, but that I am here to stay. And hopefully, just hopefully...

some of you will come join me up here too and the weight of accounting for 30% to 100% of all "blackness" in my internships, conferences and companies will finally come to a close.

But that's a paradise for another time.

Thanks for reading to the end. I'm still working on sol3, a bunch of C++ standardization efforts, and several more things!

Until next time. ♥
