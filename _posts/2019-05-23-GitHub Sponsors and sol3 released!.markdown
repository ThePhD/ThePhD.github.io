---
layout: post
title: GitHub Sponsors and sol3 released!
permalink: /sol3-released
feature-img: "assets/img/2019-05-23/logo-mona-small-wide.png"
thumbnail: "assets/img/2019-05-23/logo-mona-small-wide.png"
tags: [C++, sol2, sol3, GitHub, sponsors, 💰]
excerpt_separator: <!--more-->
---

The C++ library for fast interop to Lua is finally official, and now I can say I've done it while popping the cork for a<!--more--> new sponsorship mechanism!


# 🎊 sol3 is Released 🎊

It's been out for some time, actually, I just never formally declared it because I was worried about documentation and was trying to fix a bug with `/permissive-`. Turns out that bug is entirely unfixable at the moment, and I just have to wait for `/permissive-` to get better. In the meantime, if you want your code to be conformant to strict C++ just compile it with GCC and Clang.

But 2,087 commits in, 88 releases, and quite a bit of finagling later, sol3 is now officially out for the masses to consume. There is a SLEW of new features, renamed functionality, improved code and some updates. Most of the migration bits are covered in [this issue here](https://github.com/ThePhD/sol2/issues/776) and [this blog post](/sol3-compile-times-binary-sizes).

The benchmarks have not been fully updated yet but I expect that everything is still in-line with everything else. Some highlights:

* No more simple usertype vs. regular usertype: compilation improvements have come to the forefront and drastically improved the code, leaving only a regular `usertype` with a better, table-like interface!
* The [`dump()` function](https://github.com/ThePhD/sol2/blob/develop/examples/source/dump.cpp) is now available on all `sol::function`-like types, allowing you to serialize their byte code.
* Customization points [have been entirely overhauled](https://github.com/ThePhD/sol2/blob/develop/examples/source/customization_multiple.cpp#L7) and now you only have to write simple functions!
* Usertypes can now be `.unregister()`'d from a state.
* New [convenience type `sol::lua_value`](https://github.com/ThePhD/sol2/blob/develop/examples/source/lua_value.cpp) for easily spelling out complex types in code thanks to some feature requests [Elysian Shadow's](https://twitter.com/elysian_shadows) developer, Falco Girgis.
* Documentation has been run over a few more times and quite a few new examples have been written, based on things people found challenging over the last 8 months.
* `sol::meta_function::static_new_index` and `sol::meta_function::static_index` let you take control of the named metatable functionality rather than letting sol3 do some default behavior all the time.

Thanks so much for the donators, patrons, contributors, bug reporters and issue reproducers who helped. And thanks to the folks in Discord for holding things down while University work raged on! And, speaking of...


# 💖 GitHub Sponsors: Open Source Love 💖

I'm also proud to announce that after a run through the alpha program with some very insightful and talented folks at GitHub, I will be one of the handful of lucky developers to launch with [the GitHub Sponsors Beta](https://github.com/sponsors)! This is in addition to -- not replacing -- Patreon and a slew of other ways to show support for the sol3 project and C++ standardization in general.

Incredibly important point: **GitHub takes none of your money**.

That's right. Whereas Patreon, PayPal, etc. all take cuts of your hard earned dollary-doos to fund their operations, GitHub has brought a benefit a few scant other payment processors or donation mechanisms brings to the table.

For the first year, they will not only cover the payment processing fee, but they will **also** match your contributions to Open Source! It's pretty sweet.

Here is [my page](https://github.com/users/ThePhD/sponsorship). As for the details, the GitHub Sponsors program is pretty similar to Patreon in functionality and scope. It is a new program so it will take some time to get all the little features, but I have never met a team so communicative, welcoming and helping: all my questions were answered, my feature requests taken seriously, and my onboarding was so tremendously simple I could not have asked for an easier process. The tier system is easy to set up and use, it gives you a chance to review tiers before publicly publishing, and it gives you a nice receipt for both the people you sponsor and for the people who sponsor you. Direct integration with GitHub vastly decreases the barrier to entry for receiving continued support, making it easier than ever to have people who work hard on Open Source libraries while providing top-notch support get paid so they are not struggling.

Having recently just attended C++Now, it is amazing what being able to relax about cash when eating with everyone can do for your ability to make meaningful connections. I cannot express how much the patronage and help of people for a student like me have been. It has been a phenomenal transformation from having to pinch every penny while also providing support with a smile and continuing improvements to all of my work for 5+ years, sol3 and C++ Standards work alike.

I have no doubts that GitHub Sponsors will transform the way individuals dedicated to open source will be funded. Patreon is a good thing, but GitHub is in a unique place to make it easier than ever to support the libraries, authors and passionate folks we love in the Software Engineering discipline. Plus, given how efficient and fantastic the GitHub teams I have worked with been, I can only imagine the feature set and functionality of Sponsors to exceed most other sites as far as support for developers.

Exciting! 🎉


# 🏖️ A Summer of Busy Fun 🏖️

I have **several** things lined up to do, after I do a bit of self-care on myself after this semester. It was a rough one, and I've got another one to go before I can grasp a diploma.

There will be many more articles coming soon. And a number of trip reports! I have a **lot** to write about... but. Until then?

Keep doing your thing, and keep an eye out. 💚
