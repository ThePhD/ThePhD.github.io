---
layout: post
title: Preprocessor Embed and Language Embed - The Last Sprint
permalink: /preprocessor-embed-std-embed-the-last-spring
feature-img: "assets/img/2020-09-03/pexels-nappy-runner.jpg"
thumbnail: "assets/img/2020-09-03/pexels-nappy-runner.jpg"
tags: [C++, C, embed, preprocessor, ⌨️, 📜]
excerpt_separator: <!--more-->
---

`#embed` was reviewed by Evolution Working Group (EWG) September 2nd, 2020 and good progress was made to get the paper in shape to get it into C++23. The resulting directive, however,<!--more--> is a lot less powerful than people might have wanted.




# From the Top - What is "Embed"?

If you have not read my continual speculations/rants about the subject (or you missed the Tweets before my Twitter deactivated), I was doing a lot of work with 2 things. They were `#embed` and `std::embed`, and they were able to produce performance fixes and throughput gains of several orders of magnitude compared to stuffing incrementally larger files into binaries through a variety of means. These also provide a close to some of the oldest C and C++ frontend bugs/complaints in GCC, Clang, and MSVC including artificial string limitations and more. The gist of it is as follows:

```cpp
#include <cassert>

int main (int, char*[]) {
    const unsigned char jmp_sound[] = {
#embed <sdk/jump.wav>
    };

    // verify PCM WAV signature
    assert(jmp_sound[0] == 'R');
    assert(jmp_sound[1] == 'I');
    assert(jmp_sound[2] == 'F');
    assert(jmp_sound[3] == 'F');

    return 0;
}
```

In its simplest form, it just splats a bunch of integral values out as an _`initializer-list`_ (fancy C++ Standard Wording way to say a comma-delimited list). Note that it's just a piece of a list, and so you can add more elements between the brackets. For example, using C instead of C++:

```cpp
#include <stdio.h>

int main () {
	const unsigned char str_data[] = {
#embed "art.txt"
		, 0 // your own null termination to the data!
	};

	// just like any old string...
	puts(str_data);
	return 0;
}
```

It's a nearly 40 year old feature request with -- in general -- completely unsurprising semantics. Unfortunately, the actual details can get really hairy. What [was discussed during the Virtual EWG C++ Meeting](https://thephd.dev/_presentations/standards/C++/2020%20September/p1967/Flexible%20and%20Extensible%20Preprocessor%20Embed.html) was strictly preprocessor `#embed`. The goal was to get comfortable with a syntax and also address a big issue with providing types or other shenanigans in the preprocessor. The long and short of it is:

- The preprocessor is too simple to handle type information, and implementation experience from some vendors has made it clear that `sizeof()` support in the preprocessor is not impossible but very much a suicide mission
- The preprocessor is exactly simple enough to make vomiting out a sequence of initializer lists work nicely;
- Knowing how many bytes to pull out is a hard problem because `sizeof()` is not available in the preprocessor; thus, people have to resort to `autoconf`-style shenanigans to give `#embed` a number of bytes; and,
- This is too much effort and power to invest in the C or C++ preprocessor.

What this ultimately means is that instead of

```cpp
const hardware_entry hardware_table[]
__attribute__ (alias("my_linker_script_address"))
	= {
		#embed hardware_entry "mx22_hwell-ppc32.nv"
	};
```

and just having the compiler magically vomit out all the bits in the right place (subject to magic the compiler figures out), you would instead need to write something closer to...

```cpp
#include <span>
#include <array>

constexpr unsigned char hardware_table_data[] = {
	#embed "mx22_hwell_ppc32.nv"
};

using hardware_table_t = std::array<hardware_entry,
	sizeof(hardware_table_data) / sizeof(hardware_entry)
>;

consteval hardware_table_t
parse_hardware_table(std::span<const unsigned char>) {
	/* `memcpy`, but done at compile-time and
	fitted nicely into the structures `constexpr`.
	Maybe some `std::bit_cast<...>(...)` work on your part as well.
	*/
	return /* stuff! */;
}

const hardware_table_t hardware_table
__attribute__ (alias("my_linker_script_address"))
	= parse_hardware_table(hardware_table_data);
```


### That's... a mouthful.

Yes. But, the details of making the preprocessor aware of types before the "real frontend"/"real compiler" got to it was too hairy, and failed to gain consensus in both the C Committee and the C++ Committee. This does not sadden me: what it means is that you will have to write `consteval`/`constexpr` routines in C++ -- or pray and depend on the ability of your C compiler to constant fold things it understands -- in order to create a proper `hardware_table` array. This does not mean that there will not be work done in the future to simplify the above kind of workload in both C and C++: both Committees have individuals expressing favor towards a special kind of `bit_cast` (C++) trick, or a union-punning (C) trick to convert the elements.

This does mean that `#embed` no longer takes a "type name" collection of tokens, and instead works only in the form `#embed [optional-limit-preprocessor-expression] header-name`. This will always put out a comma-delimited list of `unsigned char`s, suitable anywhere a comma delimited list of `unsigned char`s can be. [For example, the following is allowable code](https://godbolt.org/z/67K5eq):

```cpp
int function(unsigned char a, unsigned char b, unsigned char c) {
    return a + b + c;
}

template <typename T, T a, T b, T c>
int template_function() {
    return a + b + c;
}

int main(int argc, char *argv[]) {
  constexpr unsigned char foo[] = {
#embed 3 "/etc/passwd"
  };
  int res = function(
#embed 3 "/etc/passwd"
  );
  int res2 = foo[0] + foo[1] + foo[2];
  int res3 = template_function<unsigned char,
#embed 2 "/etc/passwd"
    , foo[2]
    >();

  bool is_same = (res == res2) && (res == res3);
  return is_same ? 0 : 1;
}
```

The ability to pass this to function calls is less cool if you cannot specify your own types, but it is still nice to know that this compiles, runs, and returns the expected values in my test implementation in Clang. It solidifies the expectation that even though I implemented the internals of `#embed` using a compiler built-in for compilation speed and memory size, it is still usable in other locations. (Of course, I expect the implementation to break if I tried it in other places; I'm a librarian, not a compiler engineer.)

For the people I work with and the companies that early-adopted `#embed` and my various derpjob patches, this covers about 75% of the landscape they care about. The other folks are going to have to write `constexpr` parsers for `unsigned char` data. Sorry, I tried my best, dear reader!


### Well, if we are stuck with only one type, why not `std::byte`?

`std::byte` provides no benefits and also means this feature is incompatible with C. C does not have a `std::byte` or `std::byte`-alike type, with all the same special rules and restrictions. Having `#embed "foo.bin"` mean "list of `unsigned char`" in C and "list of `std::byte`" in C++ -- especially when we purposefully designed `std::byte` to be as user-hostile to conversions and simplicity as possible -- means that such a move would be unnecessarily mean to the C Standard.

Furthermore, while I think C++ absolutely needs `std::byte`, it's utility as a "strong type" that prevents basic math operations but requires lots of casting and pointer-punning otherwise makes its utility dubious in C. Not to mention that `std::byte` can alias, meaning it takes the same performance penalties as `char` and `unsigned char`. That is, doing byte shenanigans with `char8_t` or your own `enum class octect : unsigned char {};` can result in better assembly because the compiler will be less careful about accidentally aliasing something (but please don't do byte work with `char8_t` because we need that for Unicode, I beg you).

Nevertheless, despite being "less flexible", having a simpler `#embed` **doubles** the motivational power of a feature like `std::embed`!




# Given more Strength

One of the benefits of `std::embed` -- as a compiler built-in wrapped in Standard Template Function fashion -- is that it has no such constraints of compatibility with C or any shenanigans related to the preprocessor. This means that

```cpp
constexpr std::span<hardware_entry> my_data_span =
	[[gcc::alias("my_linker_script_address")]]
	std::embed<hardware_entry>("mx22_hwell_ppc32.nv");
```

is absolutely on the table as a way to spell "get this data and make sure it's stuck in the right place and give me a `span` of `hardware_entry`s pointing to that location while you're at it". With `#embed` unable to use special types intrinsically, `std::embed` now gets _two_ distinct advantages over `#embed`:

- Ability to use any type that is considered _`trivially-copiable`_ (`std::is_trivially_copiable_v<T>` is true); and,
- Ability to parse recursively, using the contents of the file to call `std::embed` again.

This also comes alongside the obvious benefits that are similar to `#embed`:

- Ability to apply arbitrary attributes to the function call that the implementation is free to interpret to serve the needs of high performance computing and embedded users; and,
- Always available at `consteval` time.

This was not exactly unintentional on my part. I wanted both C and C++ to have feature-parity and similar strengths. C has no way to express what `std::embed` could do in a simple way without sacrificing the ability of the compiler to perform constant folding and/or exiting any blessing given by the C Standard. I did my best to very carefully maneuver and push for a `#embed` that was exactly as strong as `std::embed` (save for the ability to be called "recursively" (e.g., can parse an arbitrary number of `#include` files inside of a shader, and even the content of those `#include`s from the shader)).

I distinctly knew this opportunity could fail, but I was okay with that failure because it only makes the motivation for a proper, `consteval`, C++-based `std::embed` even beefier. People have already begun to do things thought impossible with [patches and the available `phd::embed`](https://github.com/ThePhD/embed), from vetting typical GLSL shaders (and pulling out `location` directives) at `constexpr` time to parsing `protobuf` files. With `#embed` relegated to being simple, preprocessor-friendly and C compatible, we can now afford to really invest in greater C++ infrastructure such as an advanced `std::bit_cast` and `std::embed`.

It's unfortunate for the C Committee, but part of their charter is explicitly "we want to leave the ambitious, fun, and clever things up to C++". So, well, they'll need to use something similar to the "union trick" (this does not compile, it's just a dream):

```cpp
int main(int argc, char* argv[]) {
	double d[] = ((union { unsigned char arr[8]; double d[2]; })(
		{ #embed "foo/bar/baz.binary" }
	)).d;

	return 0;
}
```

There is more work that would have to be done here to ensure that compilers like GCC and Clang and SDCC could constant fold this if they wanted to. But, honestly, this is what C wants for itself: verbose, explicit spelling and very little "we take care of it for you" obviousness.

Nevertheless, I cannot exactly go screaming ahead with `std::embed`. There are some additional challenges for `std::embed` thanks to the last Committee direction votes it underwent at Prague, but that

is a story for another time!

Catch you on the flip side. 💚

{% include anchors.html %}
