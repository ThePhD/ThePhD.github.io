---
layout: post
title: std::embed - All the Details
permalink: /embed-the-details
feature-img: "assets/img/2019-12-18/person-soldering-chip.jpg"
thumbnail: "assets/img/2019-12-18/person-soldering-chip.jpg"
tags: [C++, C, embed, Circle, ⌨️, 📜]
excerpt_separator: <!--more-->
---

You didn't think I'd just have an irony post and not get to all the juicy technical bits, right dear reader?<!--more--> Though, to be fair, the [reddit thread](https://www.reddit.com/r/cpp/comments/ea2les/the_situation_of_stdembed/) about this exploded with a lot of commentary and exposed a lot of interesting thoughts people had about both the Standardization Process, the Committee,  `std::embed`, and C++. So, this post will be an intermingled bag of 2 things: one is all the gory details of implementing `#embed(_str)`, the [builtins behind `phd::embed(_str)`](https://github.com/ThePhD/embed), and my current experience around Circle in regards to this. Intermingled in that will be answers to a lot of question about Dependency Management, Tooling issues with `#embed`/`phd::embed`, and other tidbits.

Let's start with the fun part: speed and binary size of various approaches.



# Can Speed matter for this?

Many people pick C++ for one of two reasons: strict compatibility requirements and interop, or **speed**. These are not the only reasons, but efficiency of the final executable is an incredibly important metric to many shops around the globe. In my [last post](/full-circle-embed) I dropped some pretty disgusting timing numbers: 621 seconds to process 50 MB of data included in `#include` format, and around 16 seconds otherwise. I also claimed that Circle was not fast (but withheld the numbers while waiting for a private build of Circle from Sean Baxter) And in [another comment](https://www.reddit.com/r/cpp/comments/e5gqy3/what_is_wrong_with_stdembed_what_is_its_status/f9kyvn4), I alleged that `#embed` is garbage unless optimized.

Now that I have a final Circle build, I can present the numbers here after having given all parties a chance to optimize their code to the fullest. I also managed to optimize `#embed` so that it was no longer as memory-hungry or as bad as `xxd`.


### Methodology

The methodology is as follows. For each strategy, we simply embedded a single file of varying sizes and reported the cost to compile that file and return the second byte from the embedded data. All data was kept the same for each strategy. The final code -- sans the linker -- looked very much like so:

```cpp
int main () {
	static const unsigned char bin[] =
#embed NAME
	;

	return bin[2];
}
```

In the case of Circle, either `@embed` similarly to above or a file using the C File API (the C++ one crashes thanks to ABI shenanigans between the Circle Interpreter, libstdc++, and the runtime) with `@array` was used. For `phd::embed`, we created a magic function that sits around a built-in and is called like so:

```cpp
int main () {
	std::span<const unsigned char> bin = phd::embed<unsigned char>(NAME);

	return bin[2];
}
```

I used the `NAME` macro so I can use the flag `-DNAME=_4Mib_bin` and similar to the compiler when computing the results below. Timings were done several times in a row (despite some timings taking forever) and averages were computed on a Intel Core i7 @ 2.59 GHz with 8 Logical Cores (4 Physical) and 24.0 GB of RAM.


### Results

| Strategy           |     4 bytes    |   40 bytes    |   400 bytes   |  4 kilobytes  |
|--------------------|----------------|---------------|---------------|---------------|
| `#embed` GCC       |     0.201 s    |     0.208 s   |     0.207 s   |    0.218 s    |
| `phd::embed` GCC   |     0.709 s    |     0.724 s   |     0.711 s   |    0.715 s    |
| `objcopy` (linker) |     0.501 s    |     0.482 s   |     0.519 s   |    0.527 s    |
| Circle `@array`    |     0.353 s    |     0.359 s   |     0.361 s   |    0.361 s    |
| Circle `@embed`    |     0.199 s    |     0.208 s   |     0.204 s   |    0.368 s    |
| `xxd`-generated    |     0.225 s    |     0.215 s   |     0.237 s   |    0.247 s    |

| Strategy           |  40 kilobytes  | 400 kilobytes |  4 megabytes  |  40 megabytes |
|--------------------|----------------|---------------|---------------|---------------|
| `#embed` GCC       |     0.236 s    |    0.231 s    |     0.300 s   |     1.069 s   |
| `phd::embed` GCC   |     0.705 s    |    0.713 s    |     0.772 s   |     1.135 s   |
| `objcopy` (linker) |     0.500 s    |    0.497 s    |     0.555 s   |     2.183 s   |
| Circle `@array`    |     0.353 s    |    0.363 s    |     0.421 s   |     0.585 s   |
| Circle `@embed`    |     0.238 s    |    0.199 s    |     0.219 s   |     0.368 s   |
| `xxd`-generated    |     0.406 s    |    2.135 s    |    23.567 s   |   225.290 s   |

| Strategy           |               400 megabytes              |                1 gigabyte                 |
|--------------------|------------------------------------------|-------------------------------------------|
| `#embed` GCC       |                 9.803 s                  |                 26.383 s                  |
| `phd::embed` GCC   |                 4.170 s                  |                 11.887 s                  |
| `objcopy` (linker) |                22.654 s                  |                 58.204 s                  |
| Circle `@array`    |                 2.655 s                  |                  6.023 s                  |
| Circle `@embed`    |                 1.886 s                  |                  4.762 s                  |
| `xxd`-generated    | [OoM 😆](/assets/img/2019-12-18/xD.png)  | [OoM 😝](/assets/img/2019-12-18/xD.png)  |


To no one's surprise, `xxd`-style generated includes do not scale up to larger and larger file sizes and end up being straight garbage past 4 MB. 4 MB is the barest minimum for an uncompressed texture asset. Even for 4 MB it starts to tax developers; this can easily wreck users who try to embed multiple textures and other baseline assets -- even compressed -- into their executables.

`phd::embed` suffers a constant-time speed increase over `#embed` due to having to include several headers for `std::byte` and to call a (templated) function to embed the data. (If you call the built-in powering `phd::embed` directly and cut out all the header crap, the compilation time overhead at early numbers decreases heavily.)


### "But the Linker is Good, though!"

Unfortunately, no. No it's not.

The linker method -- which is the #1 method I was spammed about when I first shared my numbers -- does well enough as far as time goes, and does not consume undue amounts of _compiler_ memory. Unfortunately, the linker method has a serious disadvantage over the other methods and should not be used in all cases. That is, embedding data with the linker forces it to be optimizer-opaque.

In every instance of making the final executable with literally any other method, the compiler was capable enough to detect that I was only doing one `return bin_data[2];` in `int main()` and accordingly throw out all the data except that constant. Given that [compile-time JSON parsing](https://www.youtube.com/watch?v=PJwd4JLYJJY) or fast loading of large portions of the [Unicode Database](https://cor3ntin.github.io/posts/name_to_cp/) for `constexpr` use are on the horizon, it is imperative that unused or otherwise unneeded data gets discarded when not used. This may even come in handy for a potential `constexpr` implementation of [`<charconv>` using the lookup tables](https://youtu.be/4P_kbF0EbZM?t=1338) (with potentially more lookup table space required for working with 80-bit long-doubles and other shenanigans on Not MSVC). Some think it is not important that the data is discarded or optimized or carefully managed,

[but it very much is important](https://twitter.com/whitequark/status/1203448918652133376).

The linker method -- even with `-flto` and all the most aggressive optimization flags possible -- did not discard any unused `objcopy`/`ld`-dumped data. This means that embedding 4 MB of data but parsing it into a far more efficient structure as part of compile-time with template metaprogramming or `constexpr` programming no longer lets the data be optimized out at all, and instead you will always carry around the full data.

And finally, this code -- taken from P1040 and modified to recognize the harsh realities of the world -- is... just so sad:

```c
#define STRINGIZE_(x) #x
#define STRINGIZE(x) STRINGIZE_(x)

#ifdef __APPLE__
#include <mach-o/getsect.h>

#define DECLARE_LD_(LNAME) extern const unsigned char _section$__DATA__##LNAME[];
#define LD_NAME_(LNAME) _section$__DATA__##LNAME
#define LD_SIZE_(LNAME) (getsectbyLNAME("__DATA", "__" STRINGIZE(LNAME))->size)
#define DECLARE_LD(LNAME) DECLARE_LD_(LNAME)
#define LD_NAME(LNAME) LD_NAME_(LNAME)
#define LD_SIZE(LNAME) LD_SIZE_(LNAME)

#elif (defined __MINGW32__) /* mingw */

#define DECLARE_LD(LNAME)                                 \
  extern const unsigned char binary_##LNAME##_start[];    \
  extern const unsigned char binary_##LNAME##_end[];
#define LD_NAME(LNAME) binary_##LNAME##_start
#define LD_SIZE(LNAME) ((binary_##LNAME##_end) - (binary_##LNAME##_start))
#define DECLARE_LD(LNAME) DECLARE_LD_(LNAME)
#define LD_NAME(LNAME) LD_NAME_(LNAME)
#define LD_SIZE(LNAME) LD_SIZE_(LNAME)

#else /* gnu/linux ld */

#define DECLARE_LD_(LNAME)                                  \
  extern const unsigned char _binary_##LNAME##_start[];     \
  extern const unsigned char _binary_##LNAME##_end[];
#define LD_NAME_(LNAME) _binary_##LNAME##_start
#define LD_SIZE_(LNAME) ((_binary_##LNAME##_end) - (_binary_##LNAME##_start))
#define DECLARE_LD(LNAME) DECLARE_LD_(LNAME)
#define LD_NAME(LNAME) LD_NAME_(LNAME)
#define LD_SIZE(LNAME) LD_SIZE_(LNAME)
#endif

DECLARE_LD(NAME);

int main () {
	return LD_NAME(NAME)[2];
}
```

Every macro name is duplicated with the trailing underscore is to force macro expansion on all platforms. This must be written like this for compatibility with most of the major linkers I know about. God help me if I go to any more esoteric platforms! This is the stuff people say is "good enough", and I strongly disagree. Everything here works, but represents a strong failure to make a simple task -- "I want my binary data available to me and the optimizer" -- blindingly apparent.


### "👀 Yo, but that Circle 👀"

Yeah, it's the fastest!

The original numbers were nowhere near as good, but nothing scares developers into fixing up some Proof of Concept code like the potential that someone's going to scrutinize the performance. To Sean Baxter's credit, getting the theoretical maximum performance in his LLVM-based Circle in a weekend is a task that even I had a hard time doing while also dealing with Clang's `ExprConstant.cpp` leviathan, so all the hats off to him! As you can see, the latest development build of Circle scales a lot better than my hacked up GCC implementation. While getting these numbers, we ran into a bug in the interpreter when using `std::fstream` to read the data in an `@meta` context for Circle, so I had to resort to C file I/O instead in combination with `@array`.

The comparison might not be apples-to-apples because most tests are GCC and the other is LLVM-based Circle, but I did not have a Clang version prepared enough to run these tests. I will note that Clang suffers the same memory blowup problems when processing large initializer lists (a manually whitespace-shaved token-optimized `xxd`-like include file for 20 MB of binary data resulted in a 2049 MB footprint).



# Speaking of Circles...

In my last post, I pointed out my frustration with the SG7 decision. In commentary, people noted that the SG7 response -- "look for a better, generalized API" -- was perfectly reasonable. I will engage in a hot take and say

no. No it wasn't.

While I did not provide the numbers as shown above in P1040, it was no secret that parsing large initializer lists slowed down the compiler. And all of the problems of getting data in, portably, from the Linker were already bad enough that Qt -- and several other frameworks and ecosystems -- dedicated a whole tool (`qrc`) just to get around it. Developers already suffered heavily increased compile times in the presence of relatively benign uses of `constexpr` and template metaprogramming, with binary data shoveling being the worst among it. Coupled with arbitrary string literal lengths in MSVC + other compilers, and conversions from source file encoding to native encoding, portably placing data into a binary was an astounding effort. If it wasn't non-standard linker tricks, developers relied on things [like `u8` string literals to shovel binary data into their programs combined with compilers flags to treat the source file as UTF-8](https://www.reddit.com/r/cpp/comments/e5noqo/p0482r6p1423r0_nobody_uses_u8_literals/) to serve as a form of pass-through data dumping.

Programmers using string literals and non-standard linker tricks for 40 years is an indictment on C and C++'s inability to find ways to standardize existing practice for widely applicable and fundamental problems. It drives users looking for cross-platform portability to the Nonstandard Badlands™, picking up whatever trick, hack or tool gets the job done. Frequently in the Committee, we like to pretend our conservatism has no cost, that we can stall features indefinitely in search for perfection, but the reality is much different.



# Waiting has a _High_ Price

Because we have cases like the `u8` use mentioned above, it becomes a burden on the entire ecosystem when we try to do the right thing. Did we want to aggressively check `u8` string literals for well-formed UTF-8, to catch potential conversion errors or bad source-file-to-internal-compiler text transformations? Well, now we can't because somebody uses `/utf-8` and `u8` to store binary, or as a way to get out of EBCDIC land and have real, true-blue ASCII in their source.

The argument that storing binary data easily in a programming language -- whose fundamental job is to interpret binary data to do its job -- is a "niche need" is so absolutely out of touch with the day-to-day reality of wrestling with C++ it is baffling. If your language does not contain a way to handle binary data large or small -- with `std::embed`/`#embed`, a dedicated `@embed/``@array` keyword like in Circle, `slurp` from Nim, `import` files from D, or `include_bytes/str!` like Rust -- that programming language is missing a fundamental feature. That does not ultimately make the language bad or good, it just [makes it painful to work with](https://github.com/libnonius/nonius/blob/devel/include/nonius/reporters/html_reporter.h%2B%2B#L42). And given [all the complaints](https://twitter.com/oe1cxw/status/1008361214018244608), having to write `#if defined(MEDIEVAL_SADIST_VILLAIN_COMPILER)` and make _yet another workaround_ is not something we should be doing, 40 years into a "mature and production-ready" language coordinated by a large ISO body, volunteers or not.

Circle adding -- and recently optimizing -- `@array` and `@embed` proves that generalization would have brought us nowhere closer to a more sustainable and scalable future with regards to compile-time inputs; had Circle just generated a "braced initializer list" like today's tools did after reading from a file, it would have the same memory and time problems as other contemporary solutions. For a language whose [chief sin is compilation times](https://www.youtube.com/watch?v=ND-TuW0KIgg), it would behoove us to only make users use the slow and general functionality when that scalability and genericity brings serious benefits to the table. This does not mean we do not need `constexpr` file access or generalized `constexpr` I/O. But simple cases should be simple -- and optimizable -- for the 90% use case. Nothing about `std::embed` stops `constexpr std::io::input_stream` from becoming a reality. But recognizing the deliverable now -- and making it so it can be optimized by the compiler without having to perform complicated, time-consuming heuristics -- is widely benefit to the ecosystem, _today_. Not 3 or 6 or 9 years from now, when everyone gives up hope and trudges back to another 15-20 years of supporting a few more "minimum standards versions" where the basic goodies do not work (hello, 7-10 years of rolling my own `std::string_view` and `std::span` into code bases).

As further supporting evidence, trying to optimize brace initialization lists is a **hard** task. Efficient computation of something that can have 1 million integer literals all smaller than the `CHAR_BIT` that looks like the perfect binary blob, only for the 1 millionth and first integer to be computed with `some_func()` is a pathological case that would turn corner case code into real compilation nightmares.

GCC, in fact, already attempts to shrink and compress the data taken by long initializers with its internal `RANGE_EXPR` Abstract Syntax Tree node type. It's used to fold repeated expressions like `1, 2, 1, 2` and repeated numbers like `4, 4, 4, 4, 4`, as well as other sub expressions. But even after these attempts, the compiler has no extra information with which to understand large binary literals. This is primarily because while the developer's intent is "blob of binary data", the way of communicating that is "structured initialization list within braces", and the brace list is used _everywhere_ in C++ and C to mean a _lot_ of different things. `#embed` and `std::embed` provide dedicated ways to say exactly what the developer means -- "I am loading binary data!" -- and gives compiler these clues in an explicit way to get the job done.

Compile time, compiler memory, and front-end/optimizer budget should not be spent on trying to divine developer intent with internal heuristics like `RANGE_EXPR`; the language should be giving us these tools up front, to make the right decision for compiler development and for users.



# Okay, fine, but what about {Other Concern}!

Right, yes. Let's go through some of them, starting with...


### Security

At first, I was extremely concerned about this. Every time I brought up `std::embed`, somebody started in on security. It was hard to get them to articulate exactly what the security concerns would be. Opening arbitrary files? The compiler already does that. Reading in arbitrary data? Well, compile-time `fstream` or `FILE*` would behave much the same way, and those same people were asking for that too. Maybe it opens up compiler vulnerabilities...? Wait a second, compilers run a C++ parser on any old file you point at it, and it can literally pick up `/arbitrary/garbage.txt` from anywhere. You can even crash LLVM and GCC with `#include </dev/urandom>` already, but it doesn't segfault or create a security vulnerability: it just fails with Out of Memory. The more I dug into this and the more security experts I e-mailed and received responses from, I finally hit the truth.

Nothing about `#embed` or `std::embed` is more or less secure than `#include`. The biggest fear about whether or not the compiler is allowed to open or access files is just not realistic or in-tune with the reality of how the compiler works. Spoiler alert:

compilers have been reading and writing **tons** of files during builds for decades.

Temporary files, response files, Precompiled Headers, `#include` files, module maps, implementation temporaries... worries about the compiler creating/opening/reading/writing/closing stuff seemed more weird as time went on. To top that off, people have been running Python, Perl, CMake, Rust/Cargo, arbitrary C programs, and other similarly powerful tools at build time, on their very own machines as well as other people's machines with build farms. And somehow, the world has not yet imploded under security vulnerabilities.

Furthermore, source libraries and distributed binaries have never been secure or safe from an unscrupulous human being masquerading as a well-meaning developer. Are you sure that Qt does not create a global object with a constructor that launches an asynchronous request to send data up to the Qt Foundation upon program startup? Did you check all the code and verify that truth? When using a Boost library, Abseil, [DaemonSnake/unconstexpr-cpp20](https://github.com/DaemonSnake/unconstexpr-cpp20), range-v3 or literally any other piece of code, there is implicit assumption that the Boost developers or Google or DaemonSnake or Eric Niebler are not malicious. They have a Turing Complete™ language and -- somehow -- they have managed not to succumb to such a wicked temptation.

`std::embed`, `#embed`, and compile-time I/O in general changes nothing about this trust relationship at all.

Every time CMake, make, ninja, meson, and friends are executed, "untrusted code" is the dominant force driving the build and putting your software together. If reproducibility and security were concerns, the entire process should have been sandboxed from the beginning and/or regular code auditing for every line of code should have been deployed.


### Alright, but `#embed`? Macros are Gross!

I am super, duper sorry if this is you, dear reader, but I have to say that I remain wholly unconvinced by the "eww, preprocessor!" argument.

`#embed` is -- perhaps, the very first?? -- preprocessor directive that has:

- no state;
- does not affect (preprocessor) state later in program translation;
- and, flows in a single direction (preprocessor -> data available for initialization of arrays).

In other words, `#embed` is a hygienic preprocessor directive. While I understand the "eww, preprocessor" and general hatred directed at macros (well deserved, in some cases), `#embed` has none of the disadvantages that come with typical conditional compilation and macro definition/expansion shenanigans. This makes it highly suitable for the task at hand, and also strongly aids in keeping both modular and non-modular tooling from needing special handling. In fact...



# Dependency Management

This is the most important piece of criticism levied at `std::embed` and `#embed`. Dependency management in C++ is a sore topic for the typical build engineer, but we are going to focus specifically on "how do I identify all the dependencies of `#embed` and `std::embed`". Thankfully, I'm happy to report that...


### `#embed` requires no extra work.

For example, given this source file:

```cpp
#include <iterator>

int main () {
	constexpr static const char foo[] =
#embed char 3 "foo!tilde.txt"
	;

	static_assert(std::size(foo) == 3);
	static_assert(foo[0] == 'f');
	static_assert(foo[1] == 'o');
	static_assert(foo[2] == 'o');

	return 0;
}
```

with this `foo!tilde.txt`:

```
foo!~
```

when compiled with `g++ -std=c++2a -MMD main.cpp -o main.out` in the same directory, results in a `main.d` dependency file that looks like so:

```
main.out: main.cpp foo!tilde.txt
```

That's it. Everything in build systems work exactly as expected. Unfortunately...


### `std::embed` is Harder

The thing that makes `std::embed` so obscenely powerful is its ability to access files from values computed by typical `constexpr` expressions. As long as it can be (manifestly) constant evaluated, it can be done:

```cpp
#include <supercool/const_rand.h++>

int main () {
	constexpr static const std::span<const char> maybe_foo_or_bar =
		std::embed<char>(
			sc::const_rand(2) == 0 
				? "foo!tilde.txt" 
				: "bar!tidle.txt"
			, 3);
	;

	static_assert(std::size(foo) == 3);
	static_assert(maybe_foo_or_bar[0] == 'f' 
		|| maybe_foo_or_bar[0] == 'b');
	static_assert(maybe_foo_or_bar[1] == 'o' 
		|| maybe_foo_or_bar[1] == 'a');
	static_assert(maybe_foo_or_bar[2] == 'o' 
		|| maybe_foo_or_bar[2] == 'r');

	return 0;
}
```

In order to know which of `foo!tilde.txt` or `bar!tilde.txt` is used, _potentially_ every part of compilation needs to be run, save for code generation. That is, everything up through Compilation Phase 7, as determined by the Holy Standard™. Contrast that with `#embed`, which requires only up to Phase 4 preprocessing. That's a pretty big "oof". 😬

The solution here is to provide in-source hints to the compiler about where we're going to pick up our data. This was originally what [P1130](/vendor/future_cxx/papers/d1130.html) was written for, which presented a modular syntax for it:

```cpp
module bar requires "foo.txt";
module mega_bar requires { "qux.txt", "meow.jpg", "uuids.csv" };
```

I also had plans to expand the syntax usage to allow for globs, since listing every shader, icon, splash and similar is really just an excellent way to piss of every developer. So, this would work:

```cpp
module bar requires { "../../icons/*", "../assets/**" };
```

This gives tooling the ability to know what files (or directories) resources are being pulled from, without requiring to track every call to `std::embed` at constant evaluation time. There's still a question of whether it's a hard error to `std::embed` a file not under these "blessed" areas. My current feeling is that it should not be a hard error, only perhaps a warning. Others will probably feel that it should be a hard "file not found" error, even if the file exists at `"../../../../foo.txt"` but you never specified it in the `requires` clause.


### I don't like that syntax.

I care about the functionality, not the syntax. Feel free to make it `#resource requires "foo.txt" "bar/**"` if you like or anything else; suggestions welcome at all times of the day from Twitter or by e-mail or any other way you can get it to me. The syntax was not liked by EWG either, but that was before they got to see a ready form of `std::embed` or `#embed` sitting right in their laps.


# Hey, hold on, there's still some questions!

Yes, yes; I said `#embed` was garbage near the top of this lengthy detailed discussion. And it is garbage,

but I made it not-garbage with a little effort.

Originally, `#embed` performed exactly as bad as `xxd`-style brace initialization because that's exactly how I programmed it: it would encounter `#embed` and then just [vomit the data out into a bunch of tokens in braces](https://github.com/ThePhD/llvm-project/blob/1e3546cc1b7a8ea15374636fb6afe8eb9a2c6d22/clang/lib/Lex/PPDirectives.cpp#L2533). Despite a few optimizations made to the string representation for data size of the tokens (thanks, [((наб *)(войд)())()](https://nabijaczleweli.xyz/)!), it still sucked. After some pointers from `nathan`, Jakub Jelinek, Richard Biener, and others on GCC development IRC ([thank you!](https://gcc.gnu.org/ml/gcc/2019-12/msg00033.html)), I optimized it inside of GCC to not be so bad by using a special built-in I wrote called `__builtin_init_array`. When the compiler comes across the `#embed` directive it generates a built-in that writes out the built-in plus:

- file name parameter;
- null termination boolean value parameter;
- number of expected final bytes parameter;
- and, base64 encoding of the data in a string literal parameter.


### ... ... BASE64?! 🤨

Okay, now, listen. _Before_ you revoke my C++ license and go tell me to be a web developer because BASE64 DATA IN MY SEA PLUS PLUUS?!, there's a good reason for base64-encoding the data. C++ has a large sea of many players and, perhaps surprisingly, a lot of people with preprocessing tools that work on preprocessed source only or similar. (Also, it was once again an idea from [((наб *)(войд)())()](https://nabijaczleweli.xyz/)).

I had 2 problems: I needed first to have a built-in that, after `g++ -fdirectives-only` or `clang++ -frewrite-includes`, resulted in valid C++ code that could be picked up "at the other end" of, say, a `distcc` or `icecc` pipeline and still compile and be recognized by an existing compiler. The second is that there are tools which serve purely as intermediate steps and do preprocessor-based stuff only, which means someone could do the "rewrite includes/directives" flag, cram this into a bunch of intermediate tools (to downgrade some idioms the tools recognize, for example), and then pass that final result along to the real compiler.

By having the preprocessor directive produce a base64 string, I only suffered a 33% size increase of data (compared to 3/4x for the rewriting of the tokens in a _`brace-init-list`_). And, it could be passed to a distributed build system and work just fine. This is one of the reasons why `#embed` in my tests started scaling linearly with the input data around 400 MB and 1 GB and was eventually outstripped by `phd::embed` despite the C++ parsing cost of including headers and templates. There is a base64 encode and decode step to push out the data and then pick it up undisturbed after a bunch of savage tools ravage the preprocessed files. So, the web developer tricks to preserve data in hostile environments worked perfectly and provided minimal overhead! Thanks, web folks 👍.

Note that the only reason I did this is because I wanted to survive old, crusty, and unchanging tools. If this becomes a standard thing, then tools would have to respect the directive and there would be no need for suboptimal behavior and compilers could pick far better representations for rewriting the data.



# Will it be Standard?

... I mean, maybe. I'll try my best which is really all I can do. If the Standard Committees (C and C++) says no, there is always attempting to submit it as Clang, GCC and MSVC extensions and make it "de-facto" standard. But, as evidenced by compiler authors and people close to the GCC, Clang, and MSVC metal:

> ... until it's pretty certain to get into the standard, 50:50 at best... there is a much higher barrier for getting non-standard extensions in than there used to be.

> Most compiler communities (including msvc) don't really love implementing extensions like this. It undermines the committee and that, in turn, undermines one of the big things c++ has going for it.

The world for nonstandard stuff has shrank vastly, thanks to a gold rush of people putting all their favorite nonstandard things in the early compilers and [people paying the cost of backwards compatibility](https://devblogs.microsoft.com/oldnewthing/20190830-00/?p=102823) for (sometimes poorly thought out) extensions. This also makes me mildly upset with my predecessors: in the age of adding whatever nonstandard crap you wanted to a compiler or library, all of the people writing FQAs and ranting about C and C++ on comp.lang.* and in e-mails and mailing lists could have just shut up and wrote a patch. Even the so-called "Academics" that developers (game, embedded, and otherwise) like to reference with such _derision_ and _vitriol_, saying that these "perfect and pure" types are "ruining the Standard", were smarter than the Professionals.

Really.

To get around the rule limitations of their competitive programming competitions, "Academics" submitted _Policy Based Data Structures_ extensions to libstdc++ in 2004/2005. This allowed them to use hand-rolled data structures and kick untold amount of ass in programming competitions, because they had several ready-made data structures that could allow them to smoke their competition in speed just by having the mandated GCC compiler available.

Talk about 200 IQ plays!

15 years later, Red Hat maintainers can't even get rid of `#include <ext/pb_ds/...>` without vocal kickback. Can you imagine if any of the "Old Guard" of Professional developers who are currently now complaining about C and C++ just contributed to the community during the gold rush? If they had been as forward-thinking as the Academics who literally **hacked the damn standard library**, rather than sitting in their so-called ivory tower and complaining about the world? What if the old game devs who @ me and other young Committee members with complaints had done what would have likely been a weekend's worth of work at most "back in the day"? Well, just maybe I wouldn't have to write a 20 page paper (plus 2 secondary papers going to 2 different ISO Committees) and 3 blog posts and 2 separate optimized implementations with corroboration from the author of a C++ meta-language compiler to make progress on a bloody simple feature. I'd just say "standardize this existing practice".

But not in this timeline.

Instead of standing on the shoulders of giants, I have to instead struggle not to be crushed beneath the booted heel of their glib inaction, old frustrations, and callous indifference. That they should dry up the oasis of useful extensions and extended functionality while the last of us ration the water we bleed out of already dying cactuses! But it's fine, dear reader. I can't control everything -- or anything, really -- but I do understand that if I can get even just a few of these things in my hands, I know what I am capable of. I know what I can fix.

And I won't mess it up, for me or the ones who come after me.

See you in 2020. 💚
