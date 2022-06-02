---
layout: post
title: Following the River - First C Committee Meeting
permalink: /follow-the-river-wg14-ithaca-2019
feature-img: "assets/img/pexels/birds-eye-view-forest-river.jpg"
thumbnail: "assets/img/pexels/birds-eye-view-forest-river.jpg"
tags: [wg14, C, 📜️]
excerpt_separator: <!--more-->
---

If you [Follow the River, You Will Find the C](http://www.cs.columbia.edu/~jae/papers/3157-paper-v2.2-camera-final.pdf). I said this to other people and believed in it myself, but that did not mean I would embark on a spiritual journey down the riverbank myself anytime.

Or, so I thought.<!--more-->

I never thought I'd be a part of the C Standard Committee. but if I am going to fix some of these problems once and for all, I have to go back to the Beginning. It all started from a chat with Aaron Ballman. ~~The Dwarves~~Core Working Group of the C++ Standards Committee finished some papers off during a session in Köln, Germany. Aaron caught me: "Thanks for bringing in the paper on `[[nodiscard]]`! You know, C2x has attributes, and it would probably be good to get it into C too, for parity with C++..."

It was a simple conversation, and it made sense once I realized that the C Committee was _not_ dead, fixing bugs, and putting new features into the language. I was surprised that attribute syntax -- double colons and all -- made it into C too. Since this was the case, it'd create an undue burden on implementations strictly conforming to the C Standard to miss out on having a reason string. So, I packed my bags and took the original [p1301 - `[[nodiscard("should have a reason")]]`](https://wg21.link/p1301) paper from Isabella Muerte and me to the C Standards Meeting in Ithaca, New York as [ISO document N2430](https://www.open-std.org/jtc1/sc22/wg14/www/docs/n2430.pdf).




# A Different Experience

Where WG21 -- the C++ Standards Committee -- could fill an entire hotel floor with its participants, WG14 fit in a single conference room on the second floor of the Ithaca Downtown Marriot Center. I walked in and I could immediately see everyone and... well. That was it: everyone I would need to get to know, and everyone I would be working with for the next 5 days. Slightly intimidating, but after the WG21 experience of "here is an entire assembly hall and we do not have time to introduce everyone by name so let's get started with the meeting bits and if there are no seats there is an overflow room over there".

The immediate feel of the rooms is very similar to Core in WG21: almost everyone in there is fairly capable of creating wording for the C Standard. Thankfully, the syntax and reading of the C Standard is much more direct and straightforward than C++: it typically has the specification section, the semantics section, and the recommended practice section for each important clause. It takes a wide variety of implementation strategies into account, and handles a (somewhat obscene amount of) multiplicity in C compiler architectures.




# Observations from the "Outside"

I recognize by attending one of these meetings it now makes me an "insider". Sort of. I'm still sort of on the outside since I'm not working for or implementing my own C compiler or any kind of compiler (though there are people there representing themselves). I mean, sure I hacked on [GCC a little bit](https://twitter.com/strudlzrout/status/1186713609973501952), but that's it, I swear! I'm off the stuff, for good. ... What do you mean, _denial_? I am _not_ a compiler develop- don't chuckle and look at me like that, stop judging me with your eyes!

Hmph. Anyways. As a person _firmly_ on the outside, some things that really struck me:

- The C Committee values backwards compatibility, perhaps even more than the C++ Committee (albeit they are perhaps more committed to pulling the trigger on bad things, more on that later);
- For all the talks and such lauding C's stable ABI from C++ folk, there was a surprisingly high amount of discussion related to ABI and stability and struggling with new proposals that would fix real problems but might make C's ABI promises hard to keep;
- The C Committee wants forwards-compatibility with C++. C is much more respectful of C++ and its capabilities and in keeping compatibility (it's even part of their charter!);
- Implementation is Queen in WG14: there must be at least 2 implementations of something before being considered for going into the C Standard, let alone serious consideration of your paper (wishy-washy "get a feeling" papers are acceptable, so long as they are not putting anything directly into the working draft);
- There are a lot more implementations in play for consideration in WG14 than there are in WG21. C compilers counted at the meeting are in the hundreds. C++ compilers generally accounted for at the meeting are -- at most -- 5 to 7, with The Big Three (Microsoft, Clang, GCC) playing the biggest role.
- I was not the only newcomer: there was a handsome shawl-wearing man whose first meeting was the last one (London 2019), and a lady whose first meeting was the previous one as well. (Also, that shawl looked great on that guy and now I want one for myself.)

It is also a little breathtaking to see the people who voted things into early C. I got the imminent feeling that -- somehow -- I had managed to get a seat at the table with veritable goddamn giants. From CERT to IBM, from Intel to INRIA, from Linux Kernel maintainers and more... it's head-spinning the amount of security expertise and implementation experience that is sitting at the table. And this is the group _following the previous wave that has retired/took a break already_!

To think, there used to be even bigger giants in the room... Goodness Gracious. While everybody was certainly welcoming, it's hard not to feel small sitting at that kind of table. It helped that there was one familiar face from the C++ Standards Committee (thanks, Aaron), and a day or two into the week of work 2 more WG21 participants crashed in -- one of them, Paul McKenney and Niall Douglass over the phone -- which made me feel a bit more relaxed. (Not that they are any less of sheer titans, themselves! Friendly faces just ease the anxiety a bit.)

Also, there was O N E W H O L E L A D Y present, and representing a big corporation! With me heading up 100% of the blackness as per usual, and one other person there of darker-than-tan skin color (and another who joined from WG21 later in the week), well by the Gods I think percentage-wise we were actually more diverse than all the WG21 meetings I've been at, WoooOOoOooOO!

... I digress.




# So, What Happened?

There was quite a bit chewed through at this meeting. I had my own papers in there (2 of them), and there were a few from Fred Tydeman I was violently in support of. Aaron Ballman had a bunch of papers, and there was a large block dedicated to pointer lifetime and object lifetime issues (featuring Niall Douglass). These papers are the same ones C++ is struggling with and is trying to make `std::launder` and `std::start_lifetime_as` (old name: `std::bless`)) work with. A lot of this week's reading material also comes from the Absolute Unit of a Mad Lad, Jens Gustedt. Jens is pretty much pulling C from the stone ages into the modern world, recognizing the reality of progress over the last two to three decades and turning it from practice to real, standards-backed specification!


## Forward Lookin with Jens

From trying to fix `intmax_t`, proposing the actual removal of old K&R prototypes, trying to fix `time.h` functions, making `bool` a real type available in the compiler, introducing `nullptr`,  and making 2s complement integers a reality for C, Jens basically has enough papers to keep the Committee in session for eons.

It's refreshing to see someone concerned with making sure C is not just a go-between language, but is useful for getting work done with some degree of portability. While not all of his papers survived, we did a number of things that were a pretty big deal related to what Jens proposed:


### Strike down function definitions with identifier Lists (old K&R syntax)

These declarations were around from since before my birth. The gist of it is that if you wrote a declaration like this:

```cpp
void foo ();
```

You could later define it like this:

```cpp
void foo (a, b, c) 
	int a;
	long b;
	char* c;
{
	// ...
}
```

This meant that in other source files you could call `foo(1, 3L, "meow");`. But it also meant that you could call `foo(24.0, some_object);` and a diagnostic was not required (it was not ill-formed). It was undefined behavior, and frequently resulted in nasty segmentation faults or just flat out worse when the stack was not set up right. These definition lists were so horrendously foul in old codebases that [nearly every coding guideline for C -- including MISRA C -- has rules about it](https://wiki.sei.cmu.edu/confluence/display/c/DCL20-C.+Explicitly+specify+void+when+a+function+accepts+no+arguments).

It had been obsolete for 30+ years. And yet -- like lots of things in both C and C++ -- it was retained for "backwards compatibility" reasons. Eventually, code would move off of it after it was marked deprecated / obsolete. But if the last three decades have taught us anything, code that only softly warns is _never_ migrated off of. It takes hard removal to get people to move their feet, and even then implementations generally still provide backdoors.

It was a difficult decision, but we actually managed to kill it. This gave me hope, because C++ was having much the same conversation about things like `strstream`. Deprecated since C++98 but never removed, it was universally recognized as a bad idea. It was not until Peter Sommerlad fixed things up with the `stringstream` type and also worked on [a new `spanstream` type](https://wg21.link/p0448) that actual removal was discussed. We might remove it for C++23 or C++26, depending on how long until we get `spanstream` in place.

It also gives me mild hope for `std::vector<bool>`. We still keep talking about how bad it is, but the response to deprecation or removal -- even from younger C++ Committee members -- is "maybe when I retire". If C can fix one of its oldest and ugliest warts, maybe C++ can too? (Perhaps we'll have to deprecate it and wait 30 years.)


### Attempting to fix `intmax_t`

Right now, `intmax_t` is a hard ceiling that cannot ever be fixed without an ABI break. Because it is supposed to represent the maximum of all the potential integers that can exist in the implementation, it means that implementations cannot formally add `[u]int128_t`, `[u]int256_t`, or `[u]int512_t` to their "implementation defined set of integers", because that means all the functions in the C Standard that are currently ship would be ABI-broken by a change to `intmax_t`'s type.

Jens tried to design a way out of that corner, but unfortunately there were a number of problems. The entire proposal isn't dead, but it is back to the drawing board. If C figures out how to fix this, it would free up WG21 from having to languish in a hell made for it by its parent language should be adopt the latest C2x standard and also fix our own interfaces.

This also gives me _less_ hope for `std::vector<bool>`: ABI concerns kept `intmax_t` down without properly designing new types to break source, but retain binary compatibility. It's not entirely out yet, but `std::vector<bool>` is going to meet the same kind of hell (but worse, because yay templates).


### `bool`, `nullptr`, and more

Jens suggested that a lot of these keyword types and objects be added. `bool` made the most progress in the room, while `nullptr`'s semantics needed to be ironed out to ensure compatibility with C++. There's plenty of hope for `nullptr` to make it into C yet, making compatibility between the two languages all the better.



## Undefined Behavior? ... In the Preprocessor?

It's more likely than you think, and this meeting we were about to introduce even more of it into the preprocessor. There were a few implementations that could not handle parsing macros _across_ file boundaries. (Don't ask me how you end up in a place where that is even applicable to begin with; I had a hard time thinking it up myself.) The C Committee considered weakening the invariants of what was already present in the Standard: make it so macros that crossed file boundaries were undefined behavior, in order to make these crusty implementations magically conforming.

The room got a whiff of that "solution", and once we all understood the intrinsic stink of it we scrunched our noses in mild disgust. So, we voted to keep the status quo: that is, macros that span files have well-defined behavior! It sucks for that handful of implementations, but weakening the standard to make certain them conforming -- especially while using Undefined Behavior as the hammer to do it -- is a pretty bad idea in general.

Relatedly, there is a hefty amount of undefined behavior in the current preprocessor: National Bodies -- the various official voting groups apart of ISO and sub-ISO Working Groups -- filed a bunch of comments against the C++20 Committee Draft saying that Undefined Behavior in the preprocessor is a pretty garbage idea. I fully agree: no program can get off to a good start if we are invoking undefined behavior in the preprocessor. Whether it will be resolved for C++20 or C++23 (or perhaps later) remains to be seen.



## Attributes, Attributes, Attributes

Aaron Ballman led the charge here. He plastered most of C++'s useful, shared attributes and their exact syntax over to C (which was no easy feat, I gather). `[[noreturn]]` got discussed, as well as the `__has_c_attribute` preprocessor function. Some issues arose in perhaps trying to fold it down to a `__has_attribute` call and making C++ do the same so that both C and C++ share one unified set of feature test macros: each attribute returns a nonzero value when placed into the `__has(_c|_cpp)_attribute(...)` call. For C++, these values correspond to a feature testing value that is updated every time the feature is changed.

Discussion was mostly around "what happens if C++ makes a change, or C makes a change, and it isn't picked up by the other language?" or similar derps. My internal response was that we would just have to coordinate better between the two languages and hold off on feature macro testing value bumps until both working groups agreed to a value, formally or informally. I don't know how ready WG14 and WG21 are for getting cozy with one another at that level, so I essentially kept my mouth shut during this part of the conversation.

We will see how this ends up going, but honestly it should just be a matter of the two sharing a feature macro test table with one another and making sure to keep each group appraised of what they are doing: is coordination really going to be that nightmarish? Even if C or C++ add a feature into the attribute that the other doesn't want / need, it's an _attribute_: by nature the syntax can be supported, but the desired semantics entirely ignored for either language if it suits the implementer's fancy!


### `[[nodiscard]]` reason string added! ... Ish.

I did mention earlier that I put `[[nodiscard("should have a reason")]]` in the C mailing. I got to present it and it sailed through, except with one minor wording change (actually, a removal of duplicate information). This is where I found out the brutal reality of WG14:

#### WG14 loves the rules.

Where WG21 will have 300+ person meetings, drive on the veeeeery edge the ISO publication rules, keep a [running working draft in plain sight](https://eel.is/c++draft/), and other such Engineering Thuggery and Badassery, WG14 very much sticks to the rules. The draft is in a repository but kept under strict lock/key by the editor. Paper numbers are ISO document numbers. Papers can only be voted into the Standard if and only if they have no changes -- even strictly mechanical/editorial ones -- and they are identical to a document with an ISO number. As such, [the paper `[[nodiscard("should have a reason")]]`](vendor/source/C - nodiscard.html) -- with updates -- will have to wait until the March 30th Freiburg, Germany C Meeting before it is waved into the draft. It was voted that they wanted something "along the lines of N2430" into the draft, but I have to publish the next revision of the paper still and then come back around (or call in) and have it voted into the Standard.

While I understand the C Committee's need for the rules, I must say I am grateful that WG21 has a faster paper system and does not need to request ISO document numbers for their papers. This means they can make minor editorial changes to a proposal on the fly and then throw it into the Working Draft that same meeting, without having to burn a meeting cycle. I will have to make sure my papers are extra perfect before submission, otherwise I run a serious risk of being delayed a meeting!

It also means that as a student, I cannot attend anymore WG14 meetings until I register with ANSI, INCITS, or ISO as some sort of member. You get one visiting meeting for free, and then after that you need to be a paying member. I need to figure out if there might be a student discount or a reduced individual-representative fee for being a singular human attendee with no backing organization. Certain organizations and National Bodies could also designate me as an "alternative", but I do not know any such organizations willing to do that for me for the C Committee.




# Spicing Things Up

Of course, as chill and quaint as the meeting was, it was not without a little bit of fire thrown into the mix. Aaron was not just all about attributes, after all: he brought up another paper talking about the various identifier reservations WG14 makes for itself. The paper was called [What We Think We Reserve](https://www.open-std.org/jtc1/sc22/wg14/www/docs/n2409.pdf), and was a look into the pretty maddening landscape of reservations made by C. As an example from Aaron's paper:


```cpp
enum structure { // reserved
	isomorphic, // reserved
	nonisomorphic
};

void memorize_secret( // reserved
	const char* string // reserved
);

struct toxicology { // reserved
	enum condition {
		cnd_clean, // reserved
		cnd_dirty, // reserved
	} cnd;
};
```

The actual list from the C Standard is a fairly large one, and even reserves all macros beginning with `E`! Any compiler that were to pedantically warn on all reserved identifiers would probably produce a diagnostic log so large on a commercial codebase it would crash most basic log viewing panes in an IDE, or require serious pagination.

Thusly, it was brought up that, perhaps, new library functionality be prefixed with `stdc_`. We noted in the room that implementation have long since moved on from just having a limit of 7 characters for linker names. In fact, we have a minimum reported implementation size of _31_ now! Can you believe it: 2019, a whole T H I R T Y O N E `char`s, folks?!

Silliness aside, the ergonomics of having to type out `stdc_tostrp` and similar was brought up. That's a LOT of extra keystrokes, after all! And then, someone dropped 

the Namespaces word.

No, it wasn't me! But, it was easy to see the effect. The atmosphere changed immediately. Some eyebrows went up, some eyes went wide. Hands shot up to discuss the point. I got to point out that, at least, there is wide existing practice amongst current C libraries to prefix with `libabbr_decl` or similar. I also noted that macros were doing the same thing, always being `__STD_C_FOO__` these days. There was some precedent for prefixing things and following good existing C practice, so maybe we should make something easier for that?

Still, someone used the word Namespaces. A few people immediately went to "over my dead body" mode. Some went on to say that C is absolutely separate from C++, and that if someone wants namespaces they could go use C++. Someone else pointed out that we don't need mangling and linkage shenanigans, and that we should focus on simply gearing towards making it easy for users to prefix a bunch of declarations with `libfoo_`.

It was a spicy time.

We considered that we should at least potentially change some of our pattern-based reservation rules to not be so broadly overbearing. Some people are mulling over the `stdc_` stuff. I have a few ideas of my own, and maybe I'll write a paper for it! But, well...




# That's it for now.

"Wait", I hear you say, "didn't you have another paper that was kind of important?!" And, well. Yes, I did, and I presented it too. But that paper will be discussed

next time.

Toodle-oo!~

💚
