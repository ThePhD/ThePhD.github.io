---
layout: post
title: "C2y: Hitting the Ground Running"
permalink: /c2y-hitting-the-ground-running
feature-img: "/assets/img/2025/06/baseball-running-ground.jpg"
thumbnail: "/assets/img/2025/06/baseball-running-ground.jpg"
tags: [C, C standard, new, 📜]
excerpt_separator: <!--more-->
---

Surprise! Just because we released C23, doesn't mean we've stopped working on C as a whole! There is a TON of things to do, and we have absolutely been busy working on things!<!--more-->

This is a rollup of some of the more exciting things that WG14 has gotten up to in the last 10 months. A huge shoutout to Compiler Developer and Amazing Software Engineer Alex Celeste, who submitted the majority of the papers talked about in this blog and achieved GREAT SUCCESS in setting C on the path for better! We're not resting on our accomplishments for C23, as there is much to do and still yet more to accomplish! And, speaking of accomplishments, it's likely appropriate to start with _your_ accomplishments:




# `_Countof` and `countof`

[N3469](https://www.open-std.org/JTC1/SC22/WG14/www/docs/n3469.htm)

Thanks to all of you participating in our [great Managed Democracy](/the-big-array-size-survey-for-c), you have convinced WG14 to change the name from `lengthof` to `countof` for the operator name based on [your feedback](/the-big-array-size-survey-for-c-results). Previously, it had gone into C2y as `_Lengthof`/`lengthof`. When I conducted the survey, I was expecting that the consensus would match what the ARM survey showed and what most people I talked to felt: that `lengthof` was the proper name. Imagine my surprise when the survey came back and `countof` pulled ahead both in terms of raw votes in favor and was EXTREMELY ahead when using weighted votes as well!

Unfortunately, the `countof` part is still locked behind a header. That's just how C works when introducing new keywords of this nature: we have to be conservative, and the maybe in 2 to 3 standard releases we can transition it into being a serious keyword and obsolete the header. So, now, the code looks like:

```c
#include <stdcountof.h>
#include <stddef.h>

int main () {
	int arr[5];
	char arr2[20];
	const size_t n = countof(arr); // from header
	const size_t n2 = _Countof(arr2); // language keyword
	return n + n2;
}
```

This doesn't necessarily stop certain compilers from making `countof` an implementation-defined keyword anyways, but I imagine that nobody's implementation will be that brave. But, that concludes that for the forseeable future: thank you for helping us reach this decision!




# `if` Declarations

[N3356](https://www.open-std.org/JTC1/SC22/WG14/www/docs/n3356.htm)

This is a feature similar to the one deployed in C++, and one that became oft-requested for C after its utility was proven out pretty quickly in the C++ world and in C compilers that implemented C++ extensions. Fought for by Alex Celeste, this proposal mirrors the C++ version for most of its functionality for declaring a variable that's scoped to the `if` statement that can be immediately used for a test. It even comes with shortened, clean syntax that implicitly converts to `bool` to do the truth test:

```cpp
extern int fire_off(int val);

int main (int argc, char* argv[]) {
	if (int num_fired = fire_off(argc)) {
		// checks for num_fired is non-zero
	}
	else {

	}
}
```

This is equivalent to doing...

```cpp
extern int fire_off(int val);

int main (int argc, char*[]) {
	{
		int num_fired = fire_off(argc);
		if (num_fired) {
			// checks for num_fired is non-zero
		}
		else {

		}
	}
}
```

Now, occasionally you still need custom logic for the check, even with the declaration. You can do that by adding a semi-colon `;` and then putting a typical allowed conditional check afterwards. A common idiom is using `0` for the success result of an API, so you don't want to heck with `if (some_val)`, you want to use `if (!some_val)`, like so:

```cpp
#include <stdio.h>

enum err_code_t : unsigned { // C23: enum type specifiers
	err_code_ok = 0,
	err_code_invalid = 1,
	// ...
}

extern err_code_t checking_operation();

int main () {
	if (err_code_t e = checking_operation(); !e) {
		// checks for if e IS equal to zero
	}
	else {
		printf("error code: %x", (int)e);
		return 1;
	}
	return 0;
}
```

Notably, as per the "equivalent" expansion from the very first example, the `e` is available in **all** branches of the `if`/`else`/`else if` (but not outside of it). The motivation from this example is clear: getting an error code and checking if it's non-zero means you might want to do something if it actually does end up being an error, such as printing! This is mostly a usability improvement for people writing C code, and makes a few macro-based idioms easier to use and handle without things breaking irreversibly.




# New Escape Sequences (and Deprecating Octals)

[N3353](https://www.open-std.org/JTC1/SC22/WG14/www/docs/n3353.htm)

Octals have long been shown as extremely poorly designed in C and C-adjacent languages that picked up the very, VERY weird habit of leading zeros turning numbers into base-8 (octal) numbers. The justification was, as ever, "Unix Permissions!!!". Unfortunately, that's a feature for 0.001% of absolute and complete nerds, and when your programming language takes over the world for some 50 years it turns out that optimizing for something that doesn't even scale across operating systems properly becomes a **really** bad idea. This should have never been elevated to the status of a real language feature, or at the very least it should have never been "leading zeros change a number's base" which stands in stark contrast with all of mathematics and science. It doesn't even make sense, because hexadecimal -- an infinitely more useful form of bit explanation, second only to *actual* base-2 bit literals standardized in C23 -- used  the `x` from "he**x**adecimal". Was `c` from "o**c**tal" not good enough either? What about the `o`, the `t`? Even if `o` is way too visually similar, there were plenty of choices that do not end with "A raw `0` is actually an octal integer literal, actually" nerd-style trivia.

But, here we are.

Thankfully, just as K&R deprecated (but did not remove) K&R function declarations, we have finally reached a point in C where we're not going to just sit there and let old mistakes that constantly trip people up continue to slide decade after decade. Alex Celeste is here with another simple & clean proposal to get us a little bit closer to a better world. We have new escape sequences both inside of strings and a new prefix for octal numbers:

```cpp
int main () {
	const int v0 = 55; // decimal
	const int v1 = 0b00110111; // binary
	const int v2 = 0x37; // hexadecimal
	const int v3 = 0o67; // octal
	const char s0[] = "\x{37}"; // string hexadecimal
	const char s1[] = "\o{67}"; // string octal
#if 0
	// preceding line must be 0 to prevent this from compiling, because it is wrong!
	// We do not have string decimal because Octal Ruins Everything
	const char s2[] = "\55"; // byte value 45, for some fucking reason
	// We do not have string binary because \b is already bell
	const char s3[] = "\b{00110111}"; // ASCII bell, plus some random crap
#endif
	const int STOP_DOING_THIS = 067; // CEASE!
	const char FOR_THE_LOVE_OF_GOD[] = "\067"; // PLEASE!!!
	return 0;
}
```

The hope here is that, one day, `"\987"` in a string literal won't be an ugly compiler error, but a regular decimal literal. There's also the eventual hope that leader zeroes, for ALL forms of integer literals, will become irrelevant noise rather than tweaking it to suddenly become a different numeric base. The bell situation is, currently, very unfortunate, but the bell has actual uses (even if only partially as a joke) so the folks here can likely be forgiven for their hubris. Future language designers should get this stuff squared away properly and provide up-front both string and literal notations for hexadecimal, octal, decimal, and binary as their first thought. More sophisticated folks can developer more general, flexible forms, but please try not to be consistent between your strings, characters, and elsewhere: benefit from C making a dumb decision early and improve on the situation in your own language!

For now, in C, we have to sit with `070` being octal for at least 2-4 more standards cycles and then, hopefully, completely change the old behavior into decimal. This is, of course, a serious amount of cope I'm engaging in: chances are even though we finally did the right thing and obsoleted it, it'll never be fully fixed in the core language. Alas!




# Case Ranges

[N3370](https://www.open-std.org/JTC1/SC22/WG14/www/docs/n3370.htm)

This is another extension that I am unsure why it wasn't standardized before I even realized what C was as a proper programming language. It's been existence since forever and a ton of compilers use it; I also was FREQUENTLY asked about standardizing exactly this in both C and C++. While I can't help the C++ people (they'd likely put a gun to the back of the head of such a proposal to start with and instead endorse the pattern matching proposal), the C folks were happy to get this one across the finish line the moment it appeared. This one was Yet Another Banger from Alex Celeste, and it just standardizes what is existing practice:

```cpp
void foo (const char* s);

int main (int n, char* argv[]) {
	switch (n) {
	case 1:
		foo(argv[0]);
		break;
	// case 4 : // error, overlaps 2 ... 5
	//   foo ();
	//   break;
	case 2 ... 5:
		foo(argv[3]);
		break;
	case 6 ... 6: // OK (but questionable)
		foo(argv[5]);
		break;
	case 8 ... 7: // not an error, for some reason
		foo("");
		break;
	case 10 ... 4: // not an error, despite the overlap, lmao
		foo("");
		break;
	}
}
```

I'm happy that the feature is here, though as the last two cases show: it's problematic in the way it can be used. Empty ranges have to be specified by swapping the numbers: a range of a single number is just using the same value twice. It's a bit wonky the way it works in existing implementations like GCC and Clang, and the fact that it's a **fully closed** range instead of half-open means that it's problematic to access the size of an array:

```cpp
extern int index;

extern void access_arr(int* arr, int idx);

int main () {
	const int N = 30;
	int arr[N] = {};
	switch (index) {
	case 0 ... N:
		access_arr(arr, index); // ahhh damnit!
		break;
	default:
		return 1;
	}
	return 0;
}
```

This has to be written as, instead:

```cpp
extern int index;

extern void access_arr(int* arr, int idx);

int main () {
	const int N = 30;
	int arr[N] = {};
	switch (index) {
	case 0 ... N-1: // weird spelling...
		access_arr(arr, index); // but will work.
		break;
	default:
		return 1;
	}
	return 0;
}
```

This makes me not that happy about Case Ranges in C, but only because I consider this a Design Failure and not an implementation failure. The feature is **incomplete** if it doesn't work the Normal Way It Is Supposed To with things like array indices and what not. Every other language, from Kotlin to Rust, addresses this problem directly by having a second syntax: one for fully closed ranges, and another for a **half open** range. (A half-open range, one where the low number is included but the high number isn't, is how most things in C work!).

I addressed that in a technical writeup [here: Additional Half-Open Case Range Syntax](/_vendor/future_cxx/papers/C%20-%20Additional%20Half-Open%20Case%20Range%20Syntax.html). The hope is that we'll be able to move forward with something like is in this paper and go ahead and patch this hole.




# More Bit Utilities

[N3367](https://www.open-std.org/jtc1/sc22/wg14/www/docs/n3367.htm)

This is a hold over from this paper's previous iterations that didn't make the cut for C23. So, the full bit functionality is split between C23 and C2y; this paper brings a bunch of typical functions that you may or may not know about, such as:

- `uintN_t stdc_memreverse8uN(uintN_t value);` (`byteswap`/`bswap`, effectively, for some bit size `N`);
- `void stdc_memreverse8(size_t n, unsigned char ptr[static n]);` (generally-sized `byteswap` for an array);
- `generic_value_type stdc_rotate_left(generic_value_type value, generic_count_type count);`
- `generic_value_type stdc_rotate_right(generic_value_type value, generic_count_type count);`

The last two are macros, but work in the typical way as a rotate left and rotate right. There's also concrete versions for `unsigned char`, `unsigned short`, etc. etc. that use suffixes:

```cpp
unsigned char stdc_rotate_left_uc(unsigned char value, unsigned int count);
unsigned short stdc_rotate_left_us(unsigned short value, unsigned int count);
unsigned int stdc_rotate_left_ui(unsigned int value, unsigned int count);
unsigned long stdc_rotate_left_ul(unsigned long value, unsigned int count);
unsigned long long stdc_rotate_left_ull(unsigned long long value, unsigned int count);

unsigned char stdc_rotate_right_uc(unsigned char value, unsigned int count);
unsigned short stdc_rotate_right_us(unsigned short value, unsigned int count);
unsigned int stdc_rotate_right_ui(unsigned int value, unsigned int count);
unsigned long stdc_rotate_right_ul(unsigned long value, unsigned int count);
unsigned long long stdc_rotate_right_ull(unsigned long long value, unsigned int count);
```

These are in the standard now, which means C now catches up to Rust where we can use these functions in the standard and get a proper `rotl` or `rotr` without memorizing compiler intrinsics or pray that a compiler bug hasn't accidentally screwed us out of good code generation. (Not hypothetical: this stuff was VERY poorly optimized, and just writing the paper exposed deficiencies that needed to be fixed in GCC 12 and 13 and Microsoft's absolute awful quality of implementation on both x64 and ARM32 and ARM64 in this regard (thankfully, now fixed in their recent releases)).

Similarly, there's also a family of other functions for loading and storing integers in an endian-aware manner, and in both an aligned and unaligned fashion:

```cpp
uint_leastN_t stdc_load8_leuN(const unsigned char ptr[static ( N / 8)]);
uint_leastN_t stdc_load8_beuN(const unsigned char ptr[static ( N / 8)]);
uint_leastN_t stdc_load8_aligned_leuN(const unsigned char ptr[static ( N / 8)]);
uint_leastN_t stdc_load8_aligned_beuN(const unsigned char ptr[static ( N / 8)]);

int_leastN_t stdc_load8_lesN(const unsigned char ptr[static ( N / 8)]);
int_leastN_t stdc_load8_besN(const unsigned char ptr[static ( N / 8)]);
int_leastN_t stdc_load8_aligned_lesN(const unsigned char ptr[static ( N / 8)]);
int_leastN_t stdc_load8_aligned_besN(const unsigned char ptr[static ( N / 8)]);

void stdc_store8_leuN(uint_leastN_t value, unsigned char ptr[static ( N / 8)]);
void stdc_store8_beuN(uint_leastN_t value, unsigned char ptr[static ( N / 8)]);
void stdc_store8_aligned_leuN(uint_leastN_t value, unsigned char ptr[static ( N / 8)]);
void stdc_store8_aligned_beuN(uint_leastN_t value, unsigned char ptr[static ( N / 8)]);

void stdc_store8_lesN(int_leastN_t value, unsigned char ptr[static ( N / 8)]);
void stdc_store8_besN(int_leastN_t value, unsigned char ptr[static ( N / 8)]);
void stdc_store8_aligned_lesN(int_leastN_t value, unsigned char ptr[static ( N / 8)]);
void stdc_store8_aligned_besN(int_leastN_t value, unsigned char ptr[static ( N / 8)]);
```

There's big/little endian variants combined with signed/unsigned variants. If you are concerned about i.e. `int_least32_t` and `int32_t` not being the same size when you use `stdc_load8_les32`, don't: we added clauses in C23 to say that if `int32_t` exists, it must be the same type as `int_least32_t`, so you can use these functions with the exact-width integer types without being worried that things might not fit properly. You can get some significant speedups when processing data in bulk for both storing and loading such integers and get much tighter code if you know the pointer you are loading from is aligned properly for the `int64_t` or `int_least16_t` you happen to be using.

Still, a gentle word of caution for those who program fringe embedded devices: everything except the rotate left/right are gated behind `#if CHAR_BIT == 8`, so it might not exist on embedded platforms if they don't follow the type of implementation I deploy in [ztd.idk](https://github.com/soasis/idk/blob/main/include/ztd/idk/detail/bit.load_store.impl.h) that provides cross-platform, 8-bit-steady behavior. I would encourage all embedded implementations, even if they use `CHAR_BIT == 16` or `CHAR_BIT == 32` to try to use a fully bit-packed, 8-bit-aligned implementation for these things (there's a reason why I pushed to keep the name of it as `store8` and `load8`, after all).




# Labeled Breaks

[N3370](https://www.open-std.org/JTC1/SC22/WG14/www/docs/n3370.htm)

Three years ago, I mentioned in a C23 article how we did not have [a proposal for labeled loops](/ever-closer-c23-improvements#there-is-more) and that I would have preferred it over the current `break break;`, `continue break;` and `continue continue;` stuff that was in progress from Eskil Steenberg. I'm happy to report that, Yet Again, Alex Celeste crushed it by getting this contentious piece of extremely necessary functionality through into C, and even managed to get C++ to turn their eyes favorably upon this functionality.

For those who live a blissful and peaceful life, there's been a persistent problem in C-style languages because `break`, in particular, was a keyword doubly-used for both loops like `while` and `for`, as well as `switch`es:

```cpp
extern int n;

int main () {
	int x = 0;
	for (;; n -= 1) {
		switch (n) {
			default:
				// sure, do whatever
				x += n / 2;
				break;
			case 0:
				// break out of the `for` loop now
				// ...
				// ... ... ...
				// uuuuuhhhhhhhhhhhhhh
				break /*?????*/;
		}
	}
	return x;
}
```

There's nothing you can do in this situation, except set up a boolean flag, use an `if`/`else` ladder, or write a separate function and then pray you can use `return` to jump out of the nested `for`/`switch` combination. This, of course, doesn't work or scale great with triply-nested loops/`switch`es or quadruply-nested things (albeit by the time you hit quadruple nesting of anything, some folks will tell you that things have gone too far); trying to jump back to the 1st loop from the 3rd loop is an annoying task, and it gets thorny. It's a Really Fun Thing that's been a problem in the language since Forever, and every other language has various solutions for this problem.

> HEARTBREAKING: you tried to break out of a for loop inside of a switch statement in dumbass languages like C and C++. Your code fails and everyone laughs at you.
>
> ⸻ [Björkus Dorkus, May 25th, 2025](https://pony.social/@thephd/114566483437564975)

There's a better way to figure this out. And that way is Labeled Loops:

```cpp
extern int n;

int main () {
	int x = 0;
	das_loopen:
	for (;; n -= 1) {
		switch (n) {
			default:
				// sure, do whatever
				x += n / 2;
				break;
			case 0:
				// yay!!!!
				break das_loopen;
		}
	}
	return x;
}
```

You can `break SOME_LABEL;` or `continue SOME_LABEL;` out of there, and it'll work as you'd expect it to. Most other languages have this functionality, too, and it should help C developers with complicated, nested structures traverse them easily. It also dispels the heavy Moral, Social, And Technological Weight of a `goto` on Software Engineers soldiers and stay away from the scathing critiques and wary code reviewers that view it with deep suspicion. Though, if you know what you're doing? Well...

![A man eating a pizza with an obviously photoshop'd caption which reads: "(RAW GOTO) tastes so good when u ain't got a bitch in ya ear telling you it's (CONSIERED HARMFUL)".](/assets/img/2025/06/raw%20goto.jpg)

You can try it in [GCC, right now](https://godbolt.org/z/fKWb3dYMK); others are cooking up implementations in their trunks, too. There's been an (unsuccessful) attempt by [N3377](https://www.open-std.org/JTC1/SC22/WG14/www/docs/n3377.pdf) to change the location of the label in the loop after discussion in WG14, so for now it's going to stay a free-ranging label that just happens to be before the `for` or `while` or similar without any intervening statements. That means there is still room for the technological issue if reuse of labels (prevalent in macros in C), but honestly the solution for that should be getting better macro technology or a way to save a token concatenation in a macro so it can be used/reused properly. There's been some ideas around that, but nothing which has taken off (e.g., potentially having `__COUNTER__(IDENTIFIER)` as a way to make a custom incrementing counter per "`IDENTIFIER`" and then allowing to reference it without increment it with something like `__READ_COUNTER__(IDENTIFIER)`). But whether or not such things take off...

is for a future article. 💚


- Banner and Title Photo by [Pixabay, from Pexels under CC0](https://www.pexels.com/photo/baseball-game-209933/)

{% include anchors.html %}
