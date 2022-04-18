---
layout: post
title: "Conformance Should Mean Something - fputc, and Freestanding"
permalink: /conformance-should-mean-something-fputc-and-freestanding
feature-img: "/assets/img/2022/04/a - header.png"
thumbnail: "/assets/img/2022/04/a - thumbnail.png"
tags: [C, Standard]
excerpt_separator: <!--more-->
---

There is a slow-bubbling agony in my soul about this. Not because it's actually critically important or necessary but because it once again completely defies the logic of having a C Standard, a C Standard Library, or engaging in the concept of trying to "conform" to such any kind of specification. So, as per usual, I must write about it to get it out of my head:<!--more--> we need to talk about `fputc`. And, by consequence, all of the other core I/O functions in C implementations.




# The I/O Problem

We will start with an illustrating code snippet that, at first, looks completely harmless. No error handling is done, we are assuming all writes/reads are successful, flushed, etc. as error handling is not the point of the code. Here's the self-contained example:

```cpp
#include <stdio.h>

int main () {
	FILE* f = fopen("foo", "w+b");
	unsigned char c = UCHAR_MAX; // e.g. 65'535 for CHAR_BIT = 16
	unsigned char cwritten = (unsigned char)fputc(c, f);
	rewind(f);
	unsigned char cread = (unsigned char)fgetc(f);

	int ret = 0;
	if (cwritten != cread) {
		ret = 1;
	}
	else if (c != cread) {
		ret = 2;
	}
	fclose(f);
	return ret;
}
```

On 90% of most end-user facing machines, where `CHAR_BIT` — the definition of how many bits are within a single `char`, `signed char`, or `unsigned char` object — is 8, this code is nothing special. But, as the example hints at with the comment, there's more to think about. What happens when you go above the minimum-allotted size of 8 and have a `CHAR_BIT` that is either 16 or 32?

I believed that is should just write out the whole value of `c`. So with `CHAR_BIT == 16`, I would end up with 16 bits in my file and the value should be `UCHAR_MAX`, or the usual 65,535. No matter what happens, `cwritten` and `cread` should both be identical. That's covered by the fact that `fputc` always returns what it wrote to the file, and `fgetc` must always pull out the data that was written to a file. Both of these things are guaranteed by the C Standard, very explicitly and clearly, so `ret` will never be `1`. This leaves the second case: if `cread` == `cwritten`, then we only need to check if `c` is equal to one of them, not both. So, we ask the question: is `cread` the same as `c`, the value we stored in the file?

Before we get into the standard, you should know upfront that there are a lot of implementations - and users - that believe the answer is




# No.

It's a little bit weird, but yes: existing implementations of the C Standard Library — multiple, in fact — believe the answer is "no", that this program CAN return `2`, and that it's legal and right to be able to do so. This was part of my foray into many of the C and POSIX APIs, their specifications, and what they do and don't guarantee to the end-user. This one came as the biggest blow, because this throws into question much of what is and is not guaranteed by the C Standard, and whether or not we end up with **truly** portable code, or we're all just flying by the seat of our pants like Fabian makes so clear in this extremely and painfully correct pet peeve explainer:

> Pet peeve: code chock full of #ifdefs is \_not\_ "portable code". It is, at best, code that's been ported a lot. There's a big difference.
>
> the parts of your code that have platform `#ifdefs` are, by definition, exactly the parts that are \_not\_ portable
>
> — [@rygorous, February 3rd, 2020](https://twitter.com/rygorous/status/1224414412473167872)


So, are we in a situation here where we're not writing truly portable code? It's sort of a big deal, considering this is supposed to be the world's simplest, most portable I/O one could be doing. For example, consider a program which serializes a bunch of 32-bit integers and floats into an array in memory, and then proceeds to write them out:

```cpp
#include <stdio.h>

int main () {
	unsigned char data[5000];

	serialize_into(my_big_honkin_object, data, sizeof(data)/sizeof(*data));

	// moments later...
	FILE* f = fopen("my_big_honkin_data.bin", "w+b");
	fwrite(data, sizeof(*data), sizeof(data)/sizeof(*data), f);

	//

	return 0;
}
```

When I write this, I expect *all* of `my_big_honkin_object` to be in my file. If it compiles and runs, that's what should happen, no if, ands, or buts about it. Here's the literal standardese:

> ```cpp
> #include <stdio.h>
> size_t fwrite(const void* restrict ptr, size_t size, size_t nmemb,
>               FILE* restrict stream);
> ```
>
> **Description**
>
> The `fwrite` function writes, from the array pointed to by `ptr`, up to `nmemb` elements whose size is specified by size, to the stream pointed to by stream. For each object, `size` calls are made to the `fputc` function, taking the values (in order) from an array of unsigned char exactly overlaying the object …

Okay, so it will loop and call through `fputc`. This is covered under the as-if wording, so it's not like your standard library has to write **exactly** a loop of `fputc`. But, if there's a behavior where `fputc` will change each individual `unsigned char`, what kind of data are we supposed to expect in our file? Honestly, what "**things**" can an implementation of `fputc` do to the code in the first place? Well, as existing implementations today have shown...

a lot, regrettably.



## Programming Against a Non-Standard Standard

This was first called to my attention when someone casually mentioned during a discussion about Bit Utilities that some implementations reserve the right to elbow drop parts of your `unsigned char` when writing out to a stream. It was mentioned in-passing and casually, taken as assumed knowledge. It hit me like a truck, so I started sending e-mail, Direct Messages, and more to a lot of folks trying to get to the bottom of it.

What I discovered was a little more than worrying.

Not only have implementations been playing with the values when serialized into streams, it was considered ubiquitous practice on embedded chips and for strange implementations of the chip's C Standard Library, including [TI's Run-time Support Library (PDF)](https://www.ti.com/lit/an/spra757/spra757.pdf) for 2 decades and running. I got a hold of some of the folk who worked on these chips and their C library and compilers, and they confirmed that the fears presented in the linked  PDF, like "`fwrite` truncates data to 8 bits", was the real deal. Both customers of various other embedded chipsets and DSPs also reached out to me, confirming this behavior was not just niche, but prevalent in embedded programming for **decades** now.

Some implementations were, in fact, taking this code snippet:

```cpp
#include <stdio.h>

int main () {
	FILE* f = fopen("foo", "w+b");
	unsigned char c = UCHAR_MAX; // e.g. 65'535 for CHAR_BIT = 16
	unsigned char cwritten = (unsigned char)fputc(c, f);
	rewind(f);
	unsigned char cread = (unsigned char)fgetc(f);

	int ret = 0;
	if (cwritten != cread) {
		ret = 1;
	}
	else if (c != cread) {
		ret = 2;
	}
	fclose(f);
	return ret;
}
```

and returning `2` like it was just the way things should be done.

"This can't be standards-conforming!", I thought to myself. But these people — and these users — have been doing this for **over twenty years**. Assuredly, they spot-checked themselves and the standard before deciding they were just gonna do some particularly heinous things like "if you write data to a file on my chip, nothing in the standard stops me from performing a last-minute truncation of your data and writing out only 8 bits of your 16 bit or 32 bit `unsigned char`, and I will also do this to every `unsigned char` in your big fata data array too, sucker". I may be the C Project Editor, but I'm still a baby: assuredly, these industrial titans who came many, many years before me and, in some cases, programming before I was born had a better grasp of what was going on. When I publicly lamented, I was told that I shouldn't give these rabble-rousing implementations like TI a second thought, including by the developer of the popular and well-maintained musl-libc:

> If that's the case then you should be asking if it's legal, and if not why, rather than insisting it is and making up allowances to redefine things to make it so. As a starting point you need to understand that the C specification is not written in a rigorous language. It relies on making sense of natural language to draw any conclusions from it. If you insist that means you can just interpret it however you want, ...
>
> ..well, no one can stop you, but it's not a productive activity. The point is human consensus on what it means.
>
> — [@richfelker, October 21st, 2021](https://twitter.com/RichFelker/status/1451040190798123011)

I calmed myself down a little bit, after that. I mean, the guy ships **musl libc**, the preeminent reimplementation of the C Standard Library outside of glibc. Clearly, he's got a handle on it, and I shouldn't be worried about it, right?

That happened in October. But... I still couldn't quite sleep right. After all, something didn't sit right with me from the DMs and e-mails I received from other implementers and end-users who believed this **was** the correct behavior. And what struck me most was the **way** they explained themselves.




# Weasel Words

Before we get into this, there are going to be real commercial and non-commercial implementations spoken about here. Almost all of are special kinds of C implementations called "freestanding" implementations. A freestanding implementation is, informally, considered an implementation without a "host" (usually, an Operating System). It's fairly akin to what you get with `--bare-metal` flags or `-ffreestanding`, and _somewhat_ related to what comes out on the other side with Microsoft's Kernel-development flags. You know you're in "freestanding" mode by checking the `__STDC_HOSTED__` macro. It's 1 when you're on a "normal" implementation, and 0 when you're in ~~Hell~~ bare-metal mode or whatever the implementations are calling it these days.

When I first reached out to a handful of implementations, almost ALL of them had the same disclaimer placed at the top of their e-mail (so much so that I actually wondered if they were coordinating together; they weren't, but it was still eerie to get the same "waiver" of responsibility). This is a paraphrasing, but it is put in a block quote so you can read it in a voice that's not mine:

> `fread` is not part of freestanding C, and we ship a freestanding implementation, so we can do whatever we want to the I/O functions. Sorry!

And, well, I guess they're right. So, let's just pack it up and go home: clearly once you declare you're freestanding you can take non-freestanding functions and do whatever the hell you want with them, willy-nilly, no real consequences whatsoever! `fwrite` can do whatever, `fread` can do whatever, and more. It's all just smoke and mirrors and you can't ever have consistent behavior if `STDC_HOSTED` computes an integral value of 0. Alright, that's it, thanks for coming everybody, existential crisis over!

<br/>
<br/>
<br/>
<br/>
<br/>
<br/>
<br/>
<br/>
<br/>
<br/>
<br/>
<br/>
<br/>
<br/>




# ...

Are they gone? The Hyper Pedants and Strict Textualists, I mean. The people who will interpret things strictly according to the text and as long as that's satisfied they stop caring about the real humanity behind the text and the impact it has? Great. Now you and I can come down to brass tacks, Dear Reader, and get to the bottom of this. See, even if I was given the above Weasel Words as a primer, there was still more in the e-mails and messages I received back about this topic.

Some of the implementers and end-users were remorseful: they wouldn't mind the behavior for non-binary streams ("text" ones opened without the `"b"` flag for the opening mode), but for binary streams it made no sense. Others were staunch in saying they had the right to do whatever they wanted. But, and perhaps more sinisterly, there was some who believed **the wording gave them the justification to do this, even on a Hosted Implementation**. This was derived from saying that there is nothing in the `fputc` specification to stop them from doing that conversion during the write of `fputc`, since "character" here is a loose term not meant to be an actual Word of Power in the standard.

Which — and excuse my French here, I do apologize for this one, but sometimes these words are just plain necessary — blew my **fucking** mind.

No way this kind of narrow, clinching definition of `character` was allowed to be used in this way. And I was furious that it could be implied so, especially for a binary stream. §7.21.2 Streams, one paragraph reads:

> A binary stream is an ordered sequence of characters that can transparently record internal data. Data read in from a binary stream shall compare equal to the data that were earlier written out to that stream, under the same implementation. Such a stream may, however, have an implementation- defined number of null characters appended to the end of the stream.
>
> — [§7.21.2 Streams ¶3, N2731 Working Draft ISO/IEC 9899:202x, October 2021](http://www.open-std.org/jtc1/sc22/wg14/www/docs/n2731.pdf)

It has "transparently" right in the wording, assuredly that's enough. But, the only hard requirement is that data read from a binary stream must compare equal to the data written earlier to the stream, under the same implementation. Remember that `fputc` returns the "character that was written", so if a change happens *just before the write* (after the conversion to `unsigned char`), well. As long as you define "character" to mean whatever you want to here, that's just the way you can get the behavior in. Which, honestly, is deeply concern. Because there is no explicit mandate for this code snippet:

```cpp
#include <stdio.h>

int main () {
	FILE* f = fopen("foo", "w+b");
	unsigned char c = UCHAR_MAX; // e.g. 65'535 for CHAR_BIT = 16
	unsigned char cwritten = (unsigned char)fputc(c, f);
	// …
}
```

to demand that `c` is the same as `cwritten` (what is written is identical to what shall be returned, if there are no errors), implementations are free to do whatever they want. I was not the only one to notice this weasel-y reasoning in the standard, of course: someone else involved the WG14 — Working Group 14, the C Standards Committee — pointed this out to me in both an e-mail long before this popped up while I was working on [Unicode Functions for C](http://www.open-std.org/jtc1/sc22/wg14/www/docs/n2966.htm), and many years later it popped up again. It turns out this person had already climbed the ladder, asking that the Committee make a more formal definition of character so there's less room to play these kinds of games in the standard wording. And, uh, unfortunately...



## Request Denied

<center>
<img alt="A sketch of an anthropomorphic sheep with their hands pressed together, fingertips pointed up perfectly and just in front of their deeply pursed lips. Their eyes are set in a semi-wide stare out, as if in barely-restrained disbelief." src="/assets/img/2022/04/concern.png"/>
</center>

The plot got thicker here for me, because I didn't know someone else was concerned about this. This person went up the ladder and brought in a request to address the way "character" was used in the standard — especially in the Streams section of the library and the associated wording for C's string functions! — so they could properly separate out all the uses it currently has. For example, the C Standard has got at least 2 uses of "character", both using different definitions:

> **character**
>
> ⟨abstract⟩ member of a set of elements used for the organization, control, or representation of data
>
> …
>
> single-byte character
> 
> ⟨C⟩ bit representation that fits in a byte
>
> — [§3.7 and 3.7.1, N2731 Working Draft ISO/IEC 9899:202x, October 2021](http://www.open-std.org/jtc1/sc22/wg14/www/docs/n2731.pdf)

So a good fix would be to carefully annotate whether someone meant "abstract character", a "single byte", a "code unit", and so on and so forth. This would make it better to handle cases like this and others in the standard, and to at the very least make "character" a distinct Word of Power with a more precise definition (like matching it more directly to mean "*character type*" or something more formal). It would also help in this case: if we meant "abstract character", then it didn't matter what "transparently" meant in the previous wording because an implementation could do whatever they want (the input represents an "abstract character"). But, if they meant the bit representation that fits in a single byte — a single `unsigned char`, even! — then this would help us close the loop. But in a little twist that I was not fully prepared for in this story, they were told it was left *intentionally not strictly defined so people could wiggle within the definition*. … Ah.

…

… Ah.




# Hm.

That's not really all that comforting. It means that the people using the not-explicitly-specified transitive property of what is passed in after conversion to `unsigned char` to what ends being read from the file later, by the "same implementation", may be entitled to do that transformation during a binary stream. So long as they return the "character that is written", they consider themselves to be doing what they think is correct. This still brings up what the hell "transparently" means to the people operating under such an assumption, but that's English prose and not a strict definition. We would once again at the mercy of interpretation even if we *do* whip out the Merriam-Webster's Dictionary. So, still troubled by this, even after my brief interaction with the musl-libc maintainer, I decided to do what I do best: keep digging so I can get a definitive answer:

![A screenshot of an e-mail whose subject line reads: "[ Conformance ] fwrite/fputc, "character", and binary streams"](/assets/img/2022/04/fputc-e-mail.png)

As per usual, the rules are clear that I can't share what was said on the C Standard mailing list (it's okay to share the title and stuff because it's **my** e-mail, and I consent to myself sharing my own stuff, obviously). But in this case, it doesn't even matter if I can or can't share what was or was not said on the reflector: similar to when I sent my first ever e-mail to the C++ Committee E-mail List [about `std::optional`](/to-bind-and-loose-a-reference-optional#a-deafening-silence), I was met with one thing and one thing only.

[🔇 Silence! (Sound warning.)](https://static.wikia.nocookie.net/dota2_gamepedia/images/7/73/Weapons_silence.mp3/revision/latest).

But, hey, maybe I'm being unreasonable! I asked a few people to see if the e-mail was sent out, and — just like the C++ optional shenanigans — it was confirmed I had sent the e-mail out, and that people got it, and that it hadn't just been lost to The Software Stack Vortex™. And so I just sort of left the problem alone. Assuredly, someone would get back to me, soon, like within, say, maybe...

![A zoom-in on the date of the e-mail, November 5th, 2021.](/assets/img/2022/04/fputc-e-mail-date.png)

6... months?



## Mn.

Well, that's okay, because I'm not one to just sit on my hands no matter how much silence I'm met with or how much crippling depression is running through my system: I reached out to a few folks who I knew worked on MISRA, met with them, and thankfully they brought it up in their group meeting on my behalf. Even if the Committee doesn't want to / feel like commenting (and to be perfectly clear, they do not **have** to comment; it's not like I wrote a paper and nobody owes me nothin', Jack, including a response to my e-mail anyhow), at least MISRA could bring some clarity, right? They work with a **ton** of implementations, especially embedded/freestanding implementations, and so they should be able to give me good feedback. I contacted an implementer I have the utmost of faith in who attends MISRA functions, so they could bring the issue up at a meeting. They hashed it out. People for/against the code snippet above, whether `2` could be returned validly, and whether what TI's Run-Time Support Library was doing was standards-blessed behavior (ignoring any "Freestanding" weasel-ing)...

there was not consensus.

It wasn't even like there was a tiny, itty bitty holdout against the general consensus. It was an **alarmingly** split room, supposedly near half-and-half, which I suspect only wasn't half-and-half because there apparently was an uneven number and **someone** had to break the tie. This is deeply concerning that the body responsible for figuring out the dusty corners of the C standard and guaranteeing portable behavior are not sure if (a) they like what the code snippet implies and (b) whether or not they should reach in and do something about it. They're sort of on top of moving the needle to make sure we are writing high-quality code that can stand the test of time, and "`fwrite` may not portably do what you want and you need to write a wrapper function before using it every time" needs to be something they should be keen on agreeing on before we can move forward with using basic file abstractions for C. Of course, this is the human-based, common, and shared understanding I was being told about before that would lead us to Nirvana, and what I'm unfortunately finding is that it's not actually all that bound together in harmony.

Which, unfortunately, is where I begin to take issue with Rich Felker's designated approach to reading and understanding the C Standard.




# I Do Not Program on ✨ Vibes ✨

"Human consensus" is, exactly that, human. I was agitated and pushing to place more normative wording and additional examples in the C Standard because it was extremely obvious at this point that people **are** leaning on this behavior and snowballing existing practice to make this kind of behavior for `fwrite`/`fputc` be considered standards-conforming behavior. We can call it "TI's non-conforming version of not-C" or "Some DSP's not-libc" all we like, at the end of the day what matters is that there are implementations out there taking the name of POSIX and C functions and still advertising this as conforming behavior. It has leaked onto users, and those users are the ones who have to write backwards code to work on these devices, meaning that no matter the agreement I or Rich or MISRA comes to, implementations are doing what implementations have been known to do best since the EEE days of Microsoft: hold users hostage to non-portable behavior.

It would be one thing if someone was just inventing Death Station 9000, but the fact that multiple developers have reported "yeah, I had to program around that, I just assumed it was allowed there" should be enough of an indicator that this is something that needs to be addressed by the people in the driver's seat. You cannot simply toke on your pipe and casually observe and conclude that it is reasonable that no implementation should behave this way when there are, in fact, implementations behaving that way, calling themselves conforming, and have a significant chunk of embedded developers taking them up on that proposition.

Unfortunately, I have (not yet) reached a resolution to this. I have not written a paper. By the time this blog post goes out, I still will not have been given a response (and at this point I don't want one, because this needs a formal paper trail now), and I do not expect MISRA to suddenly resolve the tension either with their supposed 50/50 split on the matter. I'm also not going to stop having a quiet, slow, internal scream about these matters either, and it might take me so long to write the next paper that C23 will ship with this as the status quo and it will have to be something to take care of for C2Y/C3a, whenever it comes out...

<center>
<img alt="An anthropomorphic, smol sheep in a robe and a scarf, with beady little eyes and down-turned ears going &quot;a&quot; with their mouth open in disbelieving, mostly quiet, shocked agony." src="/assets/img/2022/04/a.png"/>
</center>

a.

But, internal screaming aside, these are moments the C Standards Committee were made for. When we need to resolve implementation tensions and make sure we give portable behavior to end-users which claim to implement our standard. Whether you believe all of those Digital Signal Processors, System-on-a-Chips, Microcontrollers and more are right to exercise their right to this interpretation or not, we should be clear that this interpretation is intended and give the end-users ways to know that much of the code they're written on "normal" implementations is compromised for these embedded environments and will do things like fundamentally shave off the high 8 / 24 bits of the 16 or 32-bit data arrays if they are not careful. At the **very** least, our users deserve a `#define __STDC_LIB_IO_WRITE_WIDTH__ (some-number)` so they can do:

```cpp
#include <limits.h>
#include <stdio.h>

#if !defined(I_ACKNOWLEDGE_MY_PLATFORM_WILL_DO_INTERESTING_THINGS) \
    || (I_ACKNOWLEDGE_MY_PLATFORM_WILL_DO_INTERESTING_THINGS == 0)

static_assert(__STDC_LIB_IO_WRITE_WIDTH__ == CHAR_BIT, "this is one hell of a spicy implementation, tread with caution");

#endif
```

But, my preferred solution would be to specify that data truncation is not a conforming trait, and force implementations - as TI has done - to document their choices and make that choice clear to end-users. Every single program which writes a whole data array to file or whatever else should not suddenly need to add special `#ifdef`'s around their code to work with what is fancied to be a conforming C implementation, freestanding or not. The human consensus is clearly not coming, and it's time for the Committee to step in so we can stop writing critical infrastructure and code on a pile of vibes.

I promise you, Dear Reader, I am going to get to the bottom of this. Either we get clear guidance and it is made clear in the Standard this was intended (even for `__STDC_HOSTED__ == 1` implementations), or we get clear guidance that this was never intended and that implementations have made a grave error. No matter which way we go, if you've written a portable program whose semantics are respected by all implementations you deserve to have a program that won't start pattern-shaving off bits.

Conformance to the C Standard should mean something. 💚

<sub>Concerned face by <a href="https://tannibeanie.wordpress.com/about-2/">Tanni H.</a>, check them out, and e-mail them to commission them for stuff too!</sub> <sub>Smol, delicate 'a' drawn by <a href="https://twitter.com/framebuffer">Framebuffer</a>, go throw them some love too!</sub>

{% include anchors.html %}
