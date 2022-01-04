---
layout: post
title: C++Now 2019 Trip Report, Extension Points, and Summer Opportunities
permalink: /c++now-2019-trip-report
feature-img: "assets/img/2019-05-23/C++Now 2019 Winners.jpg"
thumbnail: "assets/img/2019-05-23/C++Now 2019 Winners.jpg"
tags: [C++, C++Now, speaking, conferences, extension points, 🤝, 📣, 📜]
excerpt_separator: <!--more-->
---

[C++Now 2019](http://cppnow.org/) was held in scenic Aspen, Colorado again, and I geared up with two of the same roles as before but one new task:<!--more--> lead the Volunteers!



# C++ Now 2019

![A tad gloomy.](/assets/img/2019-05-23/C++Now Gloomy.jpg)

It was a bit gloomy when everyone got there, because a rain/snow storm (and potential Thunderstorm) were looming on the horizon. I was expecting to just be another Volunteer. Fill out the form, take my time slots and shifts, do the work, et cetera. But, something... weird had come together in me coming back. I was one of the few people in the Volunteer Program returning for a 2nd time, consecutively. This thrust me into a situation where I was not just to be a Volunteer, but sort of a... Volunteer Leader?

### "Our Fearless Leader!"

When I was in Rapperswil, Switzerland, Odin Holmes designated people "our fearless leader!" and it became that person's job to find a place to settle down and eat, leading the group off to one of the idyllic cafes or shops. Well, something similar happened here, where I had the prior experience and some prior knowledge and was basically told juuust before the conference got underway that I should help organize some folks. Hoh boy, I was not prepared to lead anyone. Still, using the feedback from last year's wave of volunteers, Jon Kalb, Robin Kuzmin, Michael Caisse, and Bryce Adelstein Lelbach, I worked on some of the Volunteer systems and implemented a new Admin Shadow to help make sure breaks, lunches, and the picnic went smoothly.

I'm happy to report that we got praise for our volunteer efforts! This felt quite nice, as I had never had to monitor such a thing and coordinate folks; usually I am part of the team, not leading the team! I'm certainly glad the e-mails and small amount of spreadsheet infrastructure was useful to some and that we could fluidly cover our fellow volunteers when they had critical things to get done or even when some of them fell a bit ill. Sometimes we over-subscribed on the help in certain situations, but that was partly due to having more volunteers than normal! Having more volunteers made things absolutely frictionless, and we should probably think of having more non-full-Stipend (just registration fee waived or just travel covered) volunteers for C++Now. It was of tremendous help to have more hands, and it let some volunteers relax more during the conference and spend the time doing what conferences like these are best for...


### Meeting and Greeting Folks

From seeing the forward-thinking Lisa Lippincott again (who I first met at the last C++Now) to Gašper Ažman giving me the rundown of a hot new unreleased proposal that can drastically reduce template instantiation bloat; from Daveed Vandevoorde absolutely blowing my mind with the details of compiler implementation to talking to Elias Kounen (another volunteer) who helped shape the face of the all-too popular `fmt`; C++Now is **the** place to be for experiencing not only cutting edge C++, but just plain ol' great people. Going out to Lunches and Dinners with a group of highly talented folks and getting over the massive fan factor (OH MY GOD I'M SITTING IN A CHAIR NEAR CHANDLER CARRUTH S Q U E E- wait is that THE Alisdair Meredith coming down the road oh my gosh...!) means really engaging with them as human beings, rather than just developers. Which also means that when a stranger comes up with an ADORABLE TINY PUPPY WITH A CUTE HAIRLESS BELLY OH MY GOD THAT LI'L CUTIE WAS SO DAMN ADORABL-

... Ahem. Yes, highly important networking with human beings. And adorable dogs walked by lovely people who will say Hi and have a chat with you. More technically, I don't think anything beats sitting down with Jorg Brown to hash out implementation details, or speaking with Marshall Clow and Hana Dusíková to discuss what it means to lead a successful user group. Not to mention sitting next to Odin Holmes and Eva Conti at dinner is a delight (Eva's banter is top-tier)! I don't quite talk much -- I'm usually a listener with a terrible urge to hyper focus on specific topics -- but I could listen to the people above for hours!

Which, as it turns out...


### ... I Did

Literally, for hours! I listened to the people above talk about C++ topics. From Hana's keynote on Compile-Time Regulation Expressions, to Odin Holmes on Domain-Specific Languages and the Haskell Drug, to Jeff Garland giving a full run through of `range-v3` and `std::ranges`  while also running Library in a Week every morning... goodness, there was so much to listen to! When the videos come out I am going to have to be a YouTube Junkie and get my C++ knowledge fix, with a cool drink to stave off the summer heat. [Conor Hoekstra's talks won basically all the awards](http://cppnow.org/announcements/2019/05/announcing-cpp-now-2020/#awards), but I was not able to see them! So I'll have to catch a view of them But I did more than just listen...


### I Spoke About the Future

C++Now 2019 marked the first time I spoke publicly about something not explicitly related to [sol3](https://github.com/ThePhD/sol2). During an LEWG Zombie Session at the 2018 San Diego C++ Standards meeting, Eric Fiselier and Titus Winters challenged me to provide a full sweep of extension and customization points while they were reviewing [p1132 - std::out_ptr](/_vendor/future_cxx/papers/d1132.html).

I went full-bore on the subject material, putting together a 300+ slide deck on both compile-time and run-time mechanisms for extending libraries and applications. I did not feel comfortable with everything I had done for the run-time stuff, so I threw all of that out in favor of talking specifically about the Pain Point that is extending C++ libraries in your own application code. Also, that talk would have been a 3 or 4 hour talk and I only had one and a half hours! I left the other subject material for future talks that will not necessarily by me: we definitely could use a "State of the Union" on runtime extension mechanisms near and dear to modders, security analysts, and just general end-user hackers with `LD_PRELOAD` and `mhook`. Long story short, I delivered fully on the promise of evaluating extension mechanisms for C++ library authors and their users.

The [New York C++ Developer Group Meetup](https://www.meetup.com/nyccpp/) gave me a chance to present my talk (many thanks to Arthur O'Dwyer for helping me coordinate that)! Doing dry-runs at your local user group or in front of a small group of invested friends can make all the difference, so do yourself the favor and make sure to dry-run the presentation. I can say the final product at C++Now 2019 was **far** better thanks to the suggestions and advice from the user group (go make sure to snuggle up to your local C++ user group today!). You can see the slides [here](/_presentations/sol2/C%2B%2B%20Now/2019/The%20Plan%20for%20Tomorrow%20-%20Compile-Time%20Extension%20Points%20in%20C%2B%2B.pdf). I think Titus, Eric and the C++ community in general will be pleased with the work when the video comes out. I also put [C++ Sage Matt Calabrese](https://twitter.com/CppSage/status/1126937120776630274) on the hook for his extension points paper -- [p1292](https://wg21.link/p1292) that is going to make a lot of what is bad about our current scheme SO MUCH BETTER. Seriously, if you are a compiler implementer and have free time (I know, that's a tall ask), please try to implement his new Extension Points paper and report back. And just read the paper and give general feedback, too!

I also managed to get Best Presentation, Runner Up alongside Arthur O'Dwyer's trivially relocatable talk, which is how we got in the title image at the top of this page! I really hope that this talk can serve as the basis for speaking about extension points, their benefits, and their drawbacks. This should help the C++ Community move forward into more elegant and robust forms of extension. Which brings me to one thing I missed in my presentation...


### The One Weakness of Niebloids

I didn't cover an incredibly important feature for me when I wrote [the new sol3 customization points](https://github.com/ThePhD/sol2/blob/develop/examples/source/customization_multiple.cpp). I will write about it in a future blog post, since this is not exactly the place for it! But rest assured there is an additional weakness that needs to be talked about, and to fully explain why I did not choose niebloids for my extension points!



# More to Do

![A wonderful view.](/assets/img/2019-05-23/C++Now Scenery.jpg)

Seeing as I'm also responsible for the (currently empty) [C++Now 2019 presentations repository](https://github.com/boostcon/cppnow_presentations_2019), I need to start uploading people's talks and submissions to the repository, renaming them, making sure the PDFs aren't in terribly mangled states, etc. It'll likely be my weekend (this one or next) project, so keep an eye on the repository for people's slides coming in! The talks aren't up on YouTube yet so there's not too much pressure to get it all done. If you were a speaker at C++Now 2019, make sure to send your slides in so I can put them in the repository!


### Incoming Summer Fun

I'm also pleased to announce that despite not being able to go to my original choice of internship for the Summer due to financial and housing concerns, I will instead be participating in the [Google Summer of Code 2019 under the Free Software Foundation with libstdc++](https://summerofcode.withgoogle.com/projects/#4654726870728704)! The work I will be doing will be centered around observations that the great Howard Hinnant made 7 years earlier: `vector<bool>` is a terrible name, but boy [is it _fast_](https://howardhinnant.github.io/onvectorbool.html). The formal proposal can be viewed from [this mailing list thread](https://gcc.gnu.org/ml/libstdc++/2019-02/msg00004.html). Jonathan Wakely, Ville Voutilainen, Martin Jambor, and "segher" (I only know them through contact on IRC) have been instrumental in getting me set up and off the ground to participate. I have a setup where I can compile in a Debian Sid Windows Subsystem for Linux (WSL) straight on my computer, without the need for a Linux VM! WSL2 will probably make it even easier to do this part.

It is pretty sweet that my name was recorded as ThePhD in the GSoC Project Page, so that's +1 to increasing my Developer Clout™ under that nickname. One day I'll be "professional" and use my real name (I already do for the Serious Business™ of WG21 mailings), but while I'm still a student and still having hobby fun I'll be more than happy to keep using my fun nickname!

Finally, [`boost.out_ptr` might finally be a real thing](https://github.com/ThePhD/out_ptr), maybe for Boost 1.71 or Boost 1.72. It was worked on in Library in a Week; thanks to Jeff Garland for helping me organize the in-person review session during all the fun of C++Now. Moving it forward depends on when I go ahead and submit [the mail to the mailing list asking for a review to come up](https://lists.boost.org/Archives/boost/2019/05/246248.php). I'm a bit scared: before, it was going to go into Boost 1.70 as a member of another library. Now, it needs to stand up to the cleansing, holy fire of a Boost Review.

That's all for now. There's a bunch of things I want to do to improve the C and C++ standards regarding `vector<bool>`, dynamic bit sets, Unicode support and more. I'll most certainly keep you posted!

See you fairly soon! 💚
