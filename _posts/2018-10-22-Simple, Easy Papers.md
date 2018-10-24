---
layout: post
title: Simple, Easy Papers
feature-img: "assets/img/pexels/book-glass.jpeg"
thumbnail: "assets/img/2018-10-22/San Diego Papers.jpg"
tags: [C++, future, standards, proposals, review, ⌨️, 📜]
excerpt_separator: <!--more-->
---


There are 274 papers in the San Diego mailing! But some of them hardly need committee time, but they're simple enough that there's only one answer... <!--more-->

Yes.

Reading through all of these papers takes a lot of time, and it is quite a bit more than the usual (apparently, record-breaking!) influx of papers. With such heavy volume I decided to do some pre-review of a few papers. And, in doing so, I can hopefully get everyone to reach consensus a little faster about them.

I post my sentiment at the end of each section, over the spectrum Strongly in Favor, in Favor, Neutral, Against, and Strongly Against (the same polling strategy as the working and study groups in the C++ Standards Committee).

# Starting Simple

With this many papers in the mailing, there are bound to be ones that are probably entirely inconsequential and can honestly just be answered with a quick yes/no. I've identified about 7 of these that are essentially minor and tiny enhancements that could probably be pre-gamed. Most of them are completely non-controversial and simply make the language a better form than it already is, or fix small grammar issues!

### [p1094 - Nested Inline Namespaces](https://wg21.link/p1094)

A well-reasoned and very simple paper. I reviewed r0 of this in Rapperswil on Saturday. It allows for you to specify `inline` for inline namespaces in a nested namespace identifier:

```c++
namespace foo::bar::inline v2 { ... }
```

Yes, for some reason you could not add `inline` to nested namespace identifiers before. Quite weird and surprising from the grammar. It makes sense, the motivations are clear, and it doesn't negatively impact anyone. This just needs Core Working Group (CWG) to approve it.

Strongly in Favor.

### [p1301 - `nodiscard` should have a reason](https://wg21.link/p1301)

This is another extremely simple paper. Someone handed it off to me to write, so I did: it's just about adding a attribute string token reason inside of `nodiscard`, which makes a lot of sense considering some discards are far worse than others. It also doesn't need any special wording because other attributes already do this, so it's a literal copy-paste fixup.

Strongly in Favor.


### [p1276 - `void main`](https://wg21.link/p1276)

All too often have I seen a pedant on Stack Overflow add to the comments or their answer about how wrong OP's `main` is. '_That is an extension and you're not writing conformant standard C++._', they quickly point out. It's time to retroactively fix all those answers, patch all those books, and let `main` return `void` because, let's be honest: that is kind of how the standard lets us specify it. We can leave off the return statement entirely, or only return on one branch and forget the others. For all intents and purposes, `main` feels, smells, and behaves like a `void` function in all except signature: this paper fixes that. It's time `main` started looking exactly how we specify it.

I have nothing but burning, passionate, and fiery Strongly in Favor for this paper.


### [p1272 - Byteswapping for fun&&nuf](https://wg21.link/p1272)

Yes. Or, more appropriately: _it's about damn time_. `htonl` and `ntohl` has been present since the beginning of time. That we didn't just slap it into the C++ standard and make everyone's lives easier has been tiresome. Someone finally wrote the paper, and hopefully in C++20 we will have actual standard byteswapping.

Unanimous consent, yes **please**.

### [p1278 - offsetof for the Modern Era](https://wg21.link/p1278)

Making a function out of a macro and adding features to it is only a good thing. This paper lets you specify a member object pointer to `std::offset` and get its actual offset from the type. This is pretty cool and allows generic code to do some nifty things, especially in relation to working with C APIs:

```c++
struct A {
	int a;
	int b;
	char c;
};

void main() {
	std::ptrdiff_t c_offset = std::offset(&A::c);
}
```


My only problem with the paper is that it preemptively does not mark the function `constexpr`. This is from the realization that an implementation of `std::offsetof` can just use `std::bit_cast` to implement this. `std::bit_cast` is not fully `constexpr`, and as such either `bit_cast` needs to change or this paper needs to take the leap of faith into demanding this be a requirement.

This can still be put into the Standard, even as it is. I would just have preferred it to be `constexpr` too.

In Favor!


### [p1005 - `namespace std { namespace fs = filesystem; }`](https://wg21.link/p1005)

The paper title says it all. `std::filesystem` is an enormous drag to type out. `std::fs` is much better and will have less people creating aliases or doing `using namespace std::filesystem;`.

Unanimous consent. ... That was easy, wasn't it?


### [p0330 - Literal Suffixes for size_t and ptrdiff_t](https://wg21.link/p0330)

This is a paper I picked up and wrote on behalf of a colleague. It is starting where Walter E. Brown and Rein Halbersma left off with revision 1 after an LWG meeting. This paper is old enough to have had its first revision be an N-labeled paper.

The original author got busy, but the paper was pretty much super successful and was about to be put into the Library as Library-defined literals. However, a few of the people in the room at the time realized that `size_t` and `ptrdiff_t` are types that come from the language itself, not the library, and so made a recommendation to put it into the Core Language.

I picked `z` and `uz`/`zu` to have parity with how C++ has its current built-in suffixes for literals: there is a base that represents the signed version `z` and the other that represents the unsigned version by slapping `u` before or after it. This threw out the design decisions of having separate `sz` or `s` for `size_t` and a separate `t` or `td` or `pd` for `ptrdiff_t`. I also cannot use `s` because there is another paper working on `short float` for the "half float" data type found in a lot of architectures right now. That paper would reasonably want `s`, so it makes sense not to take it. `z` has some precedence for being the size of things in C++, so this seemed like the most logical choice.

This might get bikeshed, [but other people already feel like the paper is a win](http://gcc.1065356.n8.nabble.com/C-PATCH-Implement-C-2a-P0330R2-Literal-Suffixes-for-ptrdiff-t-and-size-t-td1523832.html) and have patches lined up and ready to go!

Strongly in Favor.

### And that's it for now!

That's all the simple and easy papers. I will post about other spicier papers soon.

Until next time! ♥
