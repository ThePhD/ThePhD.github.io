---
layout: post
title: Editing the C Standard
permalink: /editing-the-c-standard
feature-img: "assets/img/2020-09-11/pexels-dom-j-typewriter-papers.jpg"
thumbnail: "assets/img/2020-09-11/pexels-dom-j-typewriter-papers.jpg"
tags: [C, 📜]
excerpt_separator: <!--more-->
---

... I did it. I survived the Paper Blitz.<!--more-->




# Huh?

For those of you who saw one of my earlier posts about [why most C implementations will purposefully blow your leg off even in the simplest of scenarios](/your-c-compiler-and-standard-library-will-not-help-you), you may have noticed that I said I became the Project Editor for C. That's a fancy way of saying I glue the standard together and produce both the Working Drafts/Working Papers (WDs/WPs), an Editor's Report, and a Diffmark of the WD against the last published WD.

The [Working Draft is here](https://drive.google.com/file/d/1IbngZ8StYVVYASd3WWuC4Uu9Xs39AF_N/view?usp=sharing) and the [Diffmarks are here](https://drive.google.com/file/d/1x4-eIBbQU3aXuORoNPSixbbl_ZKKfWfu/view?usp=sharing) until I can publish it O f f i c i a l l y through the ISO® N-Paper™ System©.

Silly symbols aside, this was a slog through 30+ papers, a few defect reports, several editorial issues, and a minor cleanup of some of the sources. There was a backlog of about 3 meetings worth of papers to integrate, give or take a few from a few missed integrations in the past and a few things early-integrated from the meetings I had to cover. Despite some pretty crazy shenanigans when I was first handling becoming the Project Editor, everything ended up turning out smoothly! For the most part, anyways: I can already smell the editorial reports for all the things I probably messed up integrating. But what is it like, editing a Standards Document? What's it like taking all those documents and turning them into a Working Draft?

![Image of a few of the papers from ](/assets/img/2020-09-11/papers.png)



# First things First - Tools

As with any project, it needs tools. This standard is built with a suite of tools any *Nix programmer will find comforting:

- `make`, for general build purposes
- `sed` and `awk` for some reserved identifier / keyword shenanigans
- `latex`, with the usual `pdflatex` and other shenanigans
- `git`, to pull a previously tagged version of the standard to attempt to create useful diffmarks

There's also a bunch of other side tools used to generate some docs and other things, but that's the core of it! As usual, nothing that discusses LaTeX would be complete without me saying the following...


### LaTeX is horseshit

Boooiiiiiiiiiiii, do I hate LaTeX. The available LaTeX distributions are piss-tier garbage, printing a line number but not tracking any file information which makes any stream of multi-compilation completely useless and requiring separate invocations of the latex compiler for each file and magic to know which file you're in. Not that that is how most people organize documents: `\input{the_file}` is the choice of developing a modular document in LaTeX land, complete with `#include`-like behavior but with 0% of the compiler Quality of Implementation.

Changing one thing of course results in a cascade of errors, warnings are split over multiple lines so as to make most error-parsing and warning-parsing regexen useless for trying to pick errors out of the literal 1 MB dump of errors, and trying to edit anything beyond the most basic of syntax is a complete slog. There is no reasonability, particular form, or dependable structure to LaTeX errors, other than "hard errors" starting with `!`.

Whitespace is not significant in the language (except when it is), people slap ad-hoc commands together to patch over LaTeX's gross inefficiencies, adding that footnote catapults the next 3 paragraphs of text to write into the margins to the right, and have you heard of our Lord and Savior `OVERFULL HBOX`?

My eternal advice to everyone is to stop writing documents in LaTeX, please. You'd get farther and faster in even Microsoft Word, and it would render better for the internet. It has math support and it doesn't poo all over the bed and scream when you want to try to add a mild margin to your author and title names, gargling with cryptic hints as to what will satisfy it enough to stop besmearing the nice linens in its dreadful output.

If your hands shake at the thought of not using BEAUTIFUL, LAYOUT-POWERFUL, HAND-BAKED LATEX on your résumé, there's templates that make your Word Documents look like LaTeX ones. Just using a different Serif-y font will probably go a long way towards that look!

... That being said, the C Standard is written in LaTeX, so that's what we're editing.


### The C Standard is Mostly Nice

Don't get me wrong: LaTeX is garbage, but the Standard I was handed is of pretty good quality for a LaTeX document. It was easy to build and edit, I did not get lost once I had all the proper things installed (I did not decide to try to figure out the "minimum required distribution" and instead just shotgun `apt get install`d `texlive-full`), and the files were orderly. Referencing things is kind of terrible but that's more of a LaTeX problem than a structure problem. I am also lucky to even be getting a LaTeX document: the standard used to be written in some God Awful Rotten Badness by the name of "TROFF". I don't know too much about it,

and I plan to keep it that way.

There's also the use of `latexdiff` and other things to produce nice diffmarks. It works out pretty decently, albeit some of the difference marks are incredibly noisy. Not that it matters: as long as it highlights the general area of changes, it produces a pretty good proxy of "places to look for things that have changed". It does ignore new added files, which is why the lists at the beginning of the Working Paper that detail the list of documents added from each meeting. Having a nice LaTeX document with some really nice organization left to me by the last editor made the next part of this article the easiest...




# Applying Your Paper to the Standard

Sweet, You Wrote a [Sick-Nasty Rad Paper](http://www.open-std.org/jtc1/sc22/wg14/www/docs/n2335.pdf) And Now It Needs To Be Put In The C Standard! So, how does it get there? Well, you don't actually write a diff to the standard. At least, not a _real_ diff: the sources to the C standard are hidden from the world to keep them safe, even when the time of Darkness descends upon us. What you write are _instructions to the project editor_ (Hi, That's Me!), and then I take your instructions and do my Best Effort™ to reflect them in the C Standard. Most of the wording gets approved by the Committee before-hand, and the edits are usually straightforward and simple. [For example](http://www.open-std.org/jtc1/sc22/wg14/www/docs/n2517.pdf):

> RECOMMENDATION: 3.4.3p4 should use a different example of undefined behavior, such as:
> > EXAMPLE An example of undefined behavior is the behavior on dereferencing a null pointer.

I then take these recommendations/instructions/suggestions and then go beat the LaTeX up on your behalf. One of the biggest benefits of this is that you don't have to know LaTeX. Or how to build the standard, or any part of that. The Project Editor is the "Layer of Indirection" between you and the actual text of the standard. This also means that I could, in theory, rewrite the entire Standard as a Microsoft Word Document, or rewrite the entire thing in restructuredText and not one of you would know the difference. Or, so that's the ideal situation, anyhow. Unfortunately, following people's Standard-editing directives not always the most straight forward...


### Instructions Unclear?!

What the hell does "3.4.3p4" mean? Well, I have to build the standard (or go look at the old one), figure out what section/paragraph you're referring to, and then fix it. Normally, the C standard evolves at such a devastatingly snail-like pace that this is normally not a problem. However, doing this was a bit tough because of 3 meetings of backlogged changes. There were lots of overlaps between papers, and also just out-and-out strange descriptions. Weird nesting in the recent C Floating Point Group's changes to the standard have been all sorts of fun to integrate.

"Delete these paragraphs, then add some here" okay, is that before or after the stuff you just asked me to destroy? Oh, I just applied a paper that deletes half of what you're asking me to edit. Uh, well, I guess we're going to have to brew up some interesting words on the fly...!

"Do these changes, and then make similar changes to the usual places" uh, what are "the usual places"? Guess it's time to break out find/replace and figure out what the usual places are! Wait, hold on, you made these changes here, but... there's identical wording somewhere else, am I supposed to edit that too...? Time to e-mail the author...

It's an interesting bucket of challenges, really. I think one of the ways to help make it so I can more reliably know what sections to edit are by adding stable tags to the standard. C++ has done this and it means when someone says "edit `[alg.any.of]`", you know where to go no matter what happens to the section and paragraph numbers. `25.6.2`... what's that, what am I doing again?

Some frustrations go beyond just the papers themselves, though.


### N2481? Nooo, you mean N2553!

One of the biggest problems in both the C and C++ Committees are history. C++ began to address this problem by using P-numbers for their papers, which are `PNNNNrXYZ` numbers that indicate both a paper uniquely and the revision of the paper (0, 1, 2, ...) in the `XYZ` part. C has no such infrastructure: every paper is submitted, officially, to ISO and is given an N-number.

How do you track revision history? You hope the author puts it in the paper title or inside the paper itself. It's basically up to the author to do this, and not all authors do it. This is a larger problem that is more strictly outside of my purview as Project Editor, but it is something that I am going to tackle anyways.

I had some inspiration thanks to the work done by Lynn Kirby, wherein Lynn created a [wg14.link website, similar to the wg21.link website](https://wg14.link). This has given me a lot of insight into what people want out of a service like this, and how I should book keep papers and their metadata. Hopefully before Summer of 2021, I will be able to unveil a new way to track these papers and keep title, author, abstract, and history information and make the Paper Submission Process far more friendly to the wider C Community!

Still, this is a lot of rambling and anymore stuff will start to get way off topic to what it means to actually edit the C Standard. Hopefully this gave you a nice little look into the world of paper/proposal wrangling!

If you have any questions, feel free to reach out to myself + other helpful folks helping me keep Project Editing straight at [wg14@soasis.org](mailto:wg14@soasis.org), or others at the newly minted [WG14 Contacts](http://www.open-std.org/jtc1/sc22/wg14/www/contacts) page. (Yes it doesn't look very modern but at least the information is up to date now; baby steps. This is a very, very old Committee...)

I'll see you in the next post!

Ta-ta for now. 💚
