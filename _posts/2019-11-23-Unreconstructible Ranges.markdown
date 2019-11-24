---
layout: post
title: The 11th Hour - Unreconstructible Ranges
permalink: /unreconstructible-ranges
feature-img: "assets/img/pexels/abandoned-building.jpg"
thumbnail: "assets/img/pexels/abandoned-building.jpg"
tags: [C++, ranges, concepts, ⌨️, 📜]
excerpt_separator: <!--more-->
---

It is with my deepest apologies that I say I have failed to make ranges reconstructible in a palatable way for C++20.<!--more-->

For those of you out of the loop, I wrote a paper called _p1664 - `reconstructible_range`s_. I then wrote a blog post about why it is a [generally good idea](/reconstructible-ranges), to generally positive reception. The goal was to put this into the Standard for C++23, and I had little to no desire to work it in for C++20. Fast forward a few days after I wrote the paper, I was told that the concept was essentially the foundation of 3 separate papers: [p1391](https://wg21.link/p1391), [p1394](https://wg21.link/p1394), and [p1739](https://wg21.link/p1739). Each of these papers wanted to give an `iterator, iterator` and `range` constructor to `string_view` and `span`, as well as take advantage of that fact in places in `std::ranges`.

The premise was that having expression-template levels of nested types and type explosions is not a good idea at all, and that if we could "flatten" the hierarchy we should do that. Thusly, it was crowned as C++20 material mostly because if we did not do this, it would become an API breaking change to later change the return types. As an example, consider the following C++20, CTADeriffic code snippets:

```cpp
std::vector vec{1, 2, 3, 4, 5};
std::span s{vec.data(), 5};
auto v = s | views::drop(1) | views::take(10)
           | views::drop(1) | views::take(10);
```

`decltype(v)` is `std::ranges::take_view<std::ranges::drop_view<std::ranges::take_view<std::ranges::drop_view<std::span<int, dynamic_extent>>>>>`. This is the status quo, and what you will end up with today. Compare to after the application of P1664:

```cpp
std::vector vec{1, 2, 3, 4, 5};
std::span s{vec.data(), 5};
auto v = s | views::drop(1) | views::take(10)
           | views::drop(1) | views::take(10);
```

`decltype(v)` is just `std::span<int>`, with P1664.



# That's Awesome!

Yeah, it is pretty magical, right? Not only do we get back the same type we put in our algorithms (`span<int>`), P1664 made it into an "exposition only" concept in the Standard. This meant that as long as you have a constructor on your type that:

- took your view's iterator and sentinel as `my_iterator, my_sentinel`;
- or, took your view's `std::ranges::subrange<my_iterator, my_sentinel>`

the ranges would always be reconstructible. This meant that everyone's ranges that obeyed the same rules for the constructors we were adding in the Standard would get the same flattening power. It also meant that we could advertise such as a way to enable your ranges, when put into Standard adaptors, to get flattened.

In the standard, this would be applied to ranges like `view::counted`, `view::take`, and `view::drop` at first: a good, limited set of ranges for C++20. As more ranges were going to be added in the Standard, I had a working list of things that would need to be reconstructible to avoid having to instantiate a `std::ranges::subrange<...>` of it and losing what type information the user gave me behind `iterator` and `sentinel` `subrange` template spew.




# Coming up with _`reconstructible-range`_

When I first conceived of reconstructible ranges, I did so because every single range in Eric Niebler's range-v3, Casey Carter's cmcstl2, Christopher DiBella's example slides and code during his nice CppCon talks, standard C++ containers, and -- after P1391 and P1394 -- most Standard Library views obeyed the concept. That is, if there was a constructor for `my_iterator, my_sentinel`, or `std::ranges::subrange<my_iterator, my_sentinel>`, it simply put the range back together. If it didn't have that constructor, it wouldn't put the range back together and the concept would report `false`. No range I found to date behaved the _wrong_ way in its presence.

It seemed like a sound premise. We passed 1 meeting with P1664, and it was approved. This was extremely important for my work on `phd.text`. I had to routinely and fundamentally take people's ranges apart with `ranges::[c]begin` and `ranges::[c]end`, which meant many of my algorithms had to unfortunately return hideous `std::ranges::subrange`s to the user for things that very much should have been reconstructible from their `begin`/`end` iterator. I used it literally everywhere as a way to ensure the user got back their `std::string_view`s and `std::text_view`s that they gave me, rather than giving them `std::ranges::subrange<std::string_view::iterator, std::string_view::iterator>` and other messy types that did not have the APIs they were used to.

It worked well.

At the Belfast C++ Standards Meeting, LEWG re-approved the design direction and sent it on its way to LWG. LWG helped fix up the wording and with a small little celebration, it ended up on the "Final Motions" page.

For those of you who do not know, while everyone can vote and move things forward in C++ Study and Working Groups during the week in the informal "straw polls", Formal Motions still must be approved by the full collection of ISO voting members and National Bodies on Saturday in Plenary. Most motions placed on the page generally have consensus and get put in with unanimous consent by this point. Occasionally, things come to a head and a formal vote is counted by the Convener.




# A Shot from the Dark

An e-mail showed up in my inbox, on Friday after the Standards Meeting had concluded, the last information straw polling session before the final Plenary session Saturday morning. It's title:

> Objection to Motion XX - reconstructible_range

To which my immediate mental reaction was:

> ... Well.
> 
> **Fuck** me.

Similarly, a Twitter DM hit me that night as well:

> ... Consider the following view, which drops the first element of the range with which it is constructed:
> 
>    struct pop_front_view {
>    	int *m_begin, *m_end;
>    
>    	pop_front_view() = default;
>    
>    	pop_front_view(int* begin, int* end)
>    	: m_begin(begin == end ? begin : begin + 1),
>    	  m_end(end)
>    	{
>    	}
>    
>    	int* begin() const { return m_begin; }
>    
>    	int* end() const { return m_end; }
>    };
> 
> The reconstructible-range machinery will erroneously use this constructor and give surprising and incorrect results...

The above range **is** reconstructible, syntactically, but it fails the semantic requirements that it "puts the range back together". I also did not have wording that required that it must be semantically reconstructible. I will be perfectly honest: I've never seen someone write a range like this, but it's totally valid code that would break reconstructible ranges.

E-mail threads started flying behind the scenes and off the Committee Reflector. Discussions started happening and circulating. And unfortunately,

there was nothing I could do.




# Just a Visitor

The Saturday Plenary is not really a place where I have any authority or power. When motions get read and objections to unanimous consent are polled, only ISO Voting Members have sway.

I am not one of those members.

This meant I could not do anything about P1664 being deferred to later, paper author or not. This is one of the dangers of being apart of the Committee but not officially under any organization's umbrella: when the **real** vote gets taken you have to rely on someone from a National Body or similar organization to take part. As given by the (soon to be observable) lack of P1664 in the C++ Working Draft,

no, P1664 didn't survive Plenary. It was deferred to Prague for re-litigation and reconsideration under a potentially new design.




# ... Now what?

Not sure. Status Quo applies: ranges aren't reconstructible and you still get a bunch of template spew. I was tasked with making an alternative design before the Prague meeting next year in February. The magic word here is "design": that means I need to go backwards, to LEWG if I get a paper in the mailing. And then I need to rewrite the wording, and go to LWG afterwards. The wording should not be too hard, but if someone objects to the design then this gets stopped all over again. So I need to publish the paper, defend a modified design early in Prague, take any changes into account, move it onto LWG's absolutely stuffed schedule, and then move it through LWG again. All in one meeting.

I am not confident in my ability to do this.

Nevertheless, the direction people pushed me in was adding an extra trait template, like:

```cpp
template <typename T>
	inline constexpr bool
	enable_reconstructible_range_v = false;

template <typename T>
	concept reconstructible-range = // exposition only
		enable_reconstructible_range_v<T>
		&& forwarding-range<T>
		&& ...; // etc. etc.
```

Yes, `false` by default. It means no range is reconstructible, and you have to opt into it. There was some mild assurance `std::basic_string_view` and `std::span` would get it turned on in the Standard. But that defied the point. Did you write your own [view type for one reason or another](https://blog.magnum.graphics/backstage/array-view-implementations/), or because the Standard didn't ship theirs yet (`unbounded_view`, `mdspan`, etc.)? Well, now you would need to `#include <ranges>` and then explicitly enable this to not have your type turn into a fountain of template instantiations if you filtered it through some of the simpler range adaptors.

It's a quick fix. A _conservative_ fix. It protects `pop_front_view` and code like it that I had overlooked. This _`reconstructible-range`_ concept change the big players were e-mailing me about and re-designing, this... this so-called 'fix'...

it **sickened** me.

Over my dead, non-National, non-ISO, non-Voting body would we force every single person to `#include <ranges>` and `template <> inline constexpr bool enable_reconstructible_range_v<MyView> = true;`. But I had to solve this for Prague. What design could I possibly come up with in time that I could re-ship in `phd.text` and verify wasn't a pile of insanity to implement and work with? Nothing about this felt good or right. I would not be able to live with a _`reconstructible`_ concept that required constant opt-in or annoying enable statements. Maybe I couldn't fix it for C++20. This _was_ supposed to be a C++23 paper. So, well...




# Taking the Nuclear Option

> Dear ...:
> 
> I withdraw P1664.

I didn't think I would ever pull something out of consideration for C++, especially something as useful as _`reconstructible-range`_. But compromising the design for a handful of ranges that exhibited special behavior was not acceptable to me. It was especially sad to me to require everyone to include/import `<ranges>`. Sure, this means `view::drop`, `view::take`, and other C++20 ranges might not get any reconstructible benefits from now until the end of time. Or -- as is currently being discussed -- maybe those benefits will be allowed for certain range adaptors, but strictly pinned to specific class types like `std::basic_string_view` and `std::span` only.

I dislike the "specific types blessed here" option as well, because it reminds me of `std::invoke`. `std::reference_wrapper` is blessed by `std::invoke` in the Standard's wording, but any other reference-wrapper type in your code base is essentially verboten. It may look, smell, and feel like a `reference_wrapper`, but it's not allowed. It's non-generic, it does not scale, and it's completely and utterly worthless for the ecosystem as a whole. Whatever they figure for `view::take` and `view::drop` has to figure it all out, but in the grand scheme of things?

Not my issue.

I embarked on this journey intending C++23, to make sure that `std::text` would be the best that it is. With P1391 and P1394 getting accepted, two important types are going to be reconstructible _in spirit_, which will serve my purposes well. Types like `unbounded_view` -- which [I talk about as an important step for proper safety in my presentation](https://youtu.be/BdUipluIf1E?t=2270) -- are not yet in the Standard, and so for my use cases I can still get the right behavior by keeping a close eye on its standardization.

I can also come up with a more flexible design for reconstructibles itself, rather than one based off of just the constructor. I have a few good ideas that will prevent having to `#include <ranges>` yet still, and which will prevent strong coupling between your type, the `<ranges>` header, and the downstream consequences of such. It will also prevent the archaic trait specialization that I see as a last resort to fixing things, and that has always caused me no shortage of problems in sol2's user interactions.

Instead of having 2-3 months to fix it before Prague, I'll have 2-3 years to do it properly for C++23. `view::drop` and `view::take` can have all the deeply-nested template spew in the world, or half-fixes.

As long as I can keep it out of `phd.text`, that's a win for me.

See you in C++23. 💚
