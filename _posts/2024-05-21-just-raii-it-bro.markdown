---
layout: post
title: "Why Not Just Do Simple C++ RAII in C?"
permalink: /just-put-raii-in-c-bro-please-bro-just-one-more-destructor-bro-cmon-im-good-for-it
feature-img: "/assets/img/2024/05/raii-banner.jpg"
thumbnail: "/assets/img/2024/05/raii-banner.jpg"
tags: [C, C++, RAII, Object Model, Effective Type, Constructors, Destructors, defer]
excerpt_separator: <!--more-->
draft: true
---

Ever since I finished publishing the "defer" paper and successfully defended it on its first go-around (it now has tentative approval to go to a Technical Specification, I just need to obtain the necessary written boilerplate to do so), an old criticism<!--more--> repeats itself frequently. Both internally to the C and C++ Standards Committee, as well as to many outside, the statement is exactly as the title implies: to implement a general-purpose undo mechanism for C, why not just make Objects with Well-Scoped, Deterministic Lifetimes and build it out of that like C++? This idiom, known as Resource Acquisition Is Initialization (RAII), is C++'s biggest bread and butter and its main claim to fame over just about every other language that grew up near it and after it (including all of the garbage collected languages such as Managed C++, D, Go, etc.). I have received no less than 5 external-to-WG14 (the formal abbreviation for the C Standards Committee) requests/asks about this, and innumerable posts internal to the C Standard mailing lists.

So, let's just get this off the table right now so I can keep referring to this post every time somebody asks:




# You ✨Cannot✨ Have "Simple RAII" in C

That's the entire premise of this article. There's a few reasons this is not possible -- some mentioned [in the `defer` paper version N3199](https://www.open-std.org/jtc1/sc22/wg14/www/docs/n3199.htm), and others that I just sort of took for granted that people would understand but do not -- and so, to clear up confusion, they will be written down here. There are two MAJOR reasons one cannot take the object-oriented semantics and syntax of RAII from C++ as-is, without jeopardizing sincere qualities about C:

- RAII is syntactically difficult to achieve in C due to the semantics imbued on those syntax constructs by C++;
- and, RAII is semantically impossible in C due to C's utterly underwhelming type/object model.

To start with, let's go over the syntax of C++, and how it achieves RAII. We will also discuss a version of RAII that uses not-C++ syntax, which would work.... at least until the second bulleted reason above dropkicks that in the face. So, let's begin:



# RAII: C++ Syntax

As a quick primer for those who are not familiar, C++ achieves its general purpose do-and-undo mechanism through the use of *constructors* and *destructors*. Constructors are function calls that are always invoked on the creation of an object, and destructors are always invoked when an object leaves scope. One can handle doing the construction and destruction manually, but we don't have to talk about such complicated cases yet. The syntax looks as follows:

```cpp
#include <cstdlib>

struct ObjectType {
	int a;
	double b;
	void* c;

	/* CONSTRUCTOR: */
	ObjectType() : a(1), b(2.2), c(malloc(30)) {

	}

	/* DESTRUCTOR: */
	~ObjectType() {
		free(c);
	}
};
```

In the above code snippet, we have a structure named `ObjectType`. It has a single constructor, that takes no arguments, and initializes all 3 of its members to some default values. It also has a destructor, which is meant to "undo" anything in the class that the programmer likes. In this case, I an using it to purposefully `free` the data that I originally `malloc`d into the member `c` during construction. Thus, when I use the class in this manner:

```cpp
#include <cstdio>

int main () {
	ObjectType thing = {};
	printf("%d %f %p", thing.a, thing.b, thing.c);
	return 0;
}
```

despite not seeing any other code in that snippet, that code will:

0. create automatic storage duration memory to put `thing` in (A.K.A. stack space for a stack variable);
1. call the constructor on that automatic storage duration memory location (A.K.A. the thing that sets those values and does `malloc`)
2. perform the `printf` call
3. prepares the `return` statement with the value of `0`
4. call the destructor on that automatic storage duration memory location (A.K.A. the thing that calls `free` to release the memory)
5. release the automatic storage duration memory (A.K.A. cleans up the stack)
6. return from the function with the value `0` being transported in whatever manner the implementation has defined

This is a fairly simple set of steps, but it's a powerful concept in C++ because no matter what happens (modulo some of the more completely bananas situations), once an object is "properly constructed" (all the data members are initialized from the `TypeName (...) : … {` list and reach the opening brace) in some region of memory, the compiler will always deterministically call the destructor at a fixed location. There is no wibbly-wobbly semantics like .NET IL finalizers or Lua `__gc` methods: the object is created, the objected is destroyed, always. (Again, we are ignoring more manual cases at the moment such as using `new`/`delete`, its array friends, or placement new & other sorts of shenanigans.) As Scott Meyers described it, this is a "powerful, general-purpose undo mechanism" and its one of the most influential concepts in deterministic, non-garbage-collected systems programming. Every other language worth being so much as spit on either employs deep garbage collection (Go, D, Java, Lua, C#, etc.) or automatic reference counting (Objective-C, Objective-C++, Swift, etc.), uses RAII (Rust with `Drop`, C++, etc.), or does absolutely nothing while saying to Go Fuck Yourself™ and kicking the developer in the shins for good measure (C, etc.).

The first problem with this, however, is a technical hangup. When C++ created their constructors, they created them with a concept called _function overloading_ in mind. This very quickly gets into the weeds of Application Binary Interfaces and other thorny issues, which are thankfully already thoroughly written about [in this expansive blog post](/to-save-c-we-must-save-abi-fixing-c-function-abi), but for the sake of brevity revisiting these concepts is helpful to understand the issue.


## Problem 0: Function Overloading

Function overloading is a technique where software engineers, in source code and syntactically, name what are at their core **two different functions** the same name. That **single** name is used as a way to referring to **two** different, distinct function calls by employing extra information, such as the number of arguments, the types of the arguments, and other clues when that single name gets used:

```cpp
// FUNCTION 0
int func (int a);
// FUNCTION 1
double func (double b);

int main () {
    int x = func(2); // calls FUNCTION 0, f(int)
    double y = func(3.3); // calls FUNCTION 1, f(double)
    return (int)(x + y);   
}
```

However, when the source code has to stop being source code and instead needs to be serialized as an actual, runnable, on-the-0s-and-1s-machine binary, linkers and loaders do not have things like compile-time "type" information and what not at-the-ready. It is too expensive to carry that information around, all the time, in perpetuity so that when someone runs a program it can differentiate between "call `f` that does stuff with an integer" versus "call `f` that does stuff with a 64-bit IEEE 754 floating point number". So, it undergoes a transformation that transforms `f(int)` or `f(double)` into something that looks like this at the assembly level:

```s
main:
        push    rbx
        mov     edi, 2
        call    _Z4funci # call FUNCTION 0
        movsd   xmm0, QWORD PTR .LC0[rip]
        mov     ebx, eax
        call    _Z4funcd # call FUNCTION 1
        movapd  xmm1, xmm0
        pxor    xmm0, xmm0
        cvtsi2sd        xmm0, ebx
        pop     rbx
        addsd   xmm0, xmm1
        cvttsd2si       eax, xmm0
        ret
.LC0:
        .long   1717986918
        .long   1074423398
```

The code looks messy because we're working with `double`s and so it generates all sorts of stuff for passing arguments and also casting it down to a 32-bit `int` for the return expression, but the 2 important lines are `call    _Z4funci` and `call    _Z4funcd`.  Believe it or not, these weird identifiers in the assembly correspond to the `func(int)` and `func(double)` identifiers in the code. This technique is called "name mangling", and it powers a huge amount of C++'s featureset. Name mangling is how, so long a argument types or number of arguments change, things like the Application Binary Interface (ABI) of function calls can be preserved. The compiler is taking the name of the function `func` and the arguments `int`/`double` and *mangling* it into the final identifier present in the binary, so that it can call the right function without having a full type system present at the machine instruction level. This has the obvious benefit that the same conceptual name can be used multiple different ways in code with different data types, mapping strongly to the "this is the algorithm, and it can work with multiple data types" idea. Thus, the compiler worries about the actual dispatch details and resolves at compile-time, which means there no runtime cost to do matching on argument count or argument types. Having it resolved at compile-time and mapped out through mangling allows it to just directly call the right code during execution. The reason this becomes important is because this is how constructors must be implemented.


## Problem 1: Member Functions

Consider the same `ObjectType` from before, but with multiple constructors:

```cpp
#include <cstdlib>

struct ObjectType {
	int a;
	double b;
	void* c;

	/* CONSTRUCTOR 0: */
	ObjectType() : a(1), b(2.2), c(malloc(30)) {

	}

	/* CONSTRUCTOR 1: */
	ObjectType(double value) : a((int)(value / 2.0)), b(value), c(malloc(30)) {

	}

	/* DESTRUCTOR: */
	~ObjectType() {
		free(c);
	}
};

#include <cstdio>

int main () {
	ObjectType x = {};
	ObjectType y = {50.0};
	printf("x: %d %f %p\n", x.a, x.b, x.c);
	printf("y: %d %f %p\n", y.a, y.b, y.c);
	return 0;
}
```

We can see the following assembly:

```s
.LC1:
	.string "x: %d %f %p\n"
.LC2:
	.string "y: %d %f %p\n"
main:
	push    r12
	push    rbp
	push    rbx
	sub     rsp, 64
	mov     rdi, rsp
	lea     rbp, [rsp+32]
	mov     rbx, rsp
	call    _ZN10ObjectTypeC1Ev
	movsd   xmm0, QWORD PTR .LC0[rip]
	mov     rdi, rbp
	call    _ZN10ObjectTypeC1Ed
	mov     rdx, QWORD PTR [rsp+16]
	movsd   xmm0, QWORD PTR [rsp+8]
	mov     edi, OFFSET FLAT:.LC1
	mov     eax, 1
	mov     esi, DWORD PTR [rsp]
	call    printf
	mov     rdx, QWORD PTR [rsp+48]
	movsd   xmm0, QWORD PTR [rsp+40]
	mov     edi, OFFSET FLAT:.LC2
	mov     eax, 1
	mov     esi, DWORD PTR [rsp+32]
	call    printf
	mov     rdi, rbp
	call    _ZN10ObjectTypeD1Ev
	mov     rdi, rsp
	call    _ZN10ObjectTypeD1Ev
	add     rsp, 64
	xor     eax, eax
	pop     rbx
	pop     rbp
	pop     r12
	ret
	mov     r12, rax
	jmp     .L3
	mov     r12, rax
	jmp     .L2
main.cold:
.L2:
	mov     rdi, rbp
	call    _ZN10ObjectTypeD1Ev
.L3:
	mov     rdi, rbx
	call    _ZN10ObjectTypeD1Ev
	mov     rdi, r12
	call    _Unwind_Resume
.LC0:
	.long   0
	.long   1078525952 
```

Again, we notice in particular the use of these special, mangled identifiers for the `call` instructions: `call    _ZN10ObjectTypeC1Ev`, `call    _ZN10ObjectTypeC1Ed`, and `call    _ZN10ObjectTypeD1Ev`. It has the name of the type (`…10ObjectType…`) in it this time, but more or less just mangles it out. This is where the heart of our problems lie. If C wants to steal C++'s syntax for RAII, and C wants to be able to share (header file) source code that enjoys simple RAII objects, every single C implementation needs to implement a Name Mangler compatible with C++ for the platforms they target. And how hard could that possibly be?

![](/assets/img/2024/05/ten-dollars-arrested-development.gif)



## Hm.

Here are some name manglings for the one argument `ObjectType` constructor:

- `_ZN10ObjectTypeC1Ed` (GCC/Clang on Linux; x86-64, ARM, ARM64, and i686)
- `??0ObjectType@@QEAA@N@Z` (MSVC; x86-64, ARM64)
- `??0ObjectType@@QAE@N@Z` (MSVC; i686)

That's three different name manglings for only a handful of platforms! And while some name manglers are partially documented or at least provided as a library so that it can be built upon, the name manglers for others are not only utterly undocumented but completely inscrutable. So much so that on some platforms (like MSVC on any architecture), certain name manglings are not guaranteed to be 1:1 and can infact "demangle" into multiple different plausible entities. If an implementation gets the name mangling wrong, well, that's just a damn shame for the end user who has to deal with it! Of course, nobody's claiming that name mangling is an unsolvable problem; it is readily solved in codebases such as Clang and GCC. But, it is worth noting that, as C's specification stands now, there is no requirement to mangle any functions.

This is both a blessing, and a curse. The former because functions that users write are pretty much 1:1 when they are written under a C compiler. If a functioned is named `glorbinflorbin` in C, the name that shows up in the binary is `glorbinflorbin` with maybe some extra underscores added in places somewhere on certain implementations. But, the latter comes in to play for precisely this reason: if there is no name mangling performed that considers things such as related enclosing member object, argument types, and similar, then it is impossible to have even mildly useful features that can do things like avoid name clashes a function prototype is generated with the wrong types. It is, in fact, the primary reason that C ends up in interesting problems when using `typedef`s inside of its function declarations. Even if the `typedef`s change, the function names do not because there is **no concept** of "member functions" or "function overloading" or anything like that. It's why [the `intmax_t` problem is such an annoying one](https://thephd.dev/intmax_t-hell-c++-c).



## What Does This Have To Do With RAII?

Well, the devil is in these sorts of details. In order to introduce nominal support for something like constructors, name mangling (or something that allows the user to control how names come out on the other side) need to be made manifest in C. If name mangling is chosen as the implementation choice and a syntax identical to C++ is chosen, the implementation becomes responsible for furnishing a name mangler. And, because people are (usually) not trying to be evil, there should be ABI compatibility between the C implementation's name mangler and C++'s name mangler so that code written with constructors in one language interoperate just fine with the other, without requiiring `extern "C"` to be placed on every constructor. (Right now, `extern "C"` is not legal to place on any member function in any C++ implementation.)

The reason this is desirable is obvious: header code could be shared between the languages, which makes sense in a world where "just steal C++'s constructors and destructors" is the overall design decision for C. But this is very much a nonstarter implementation reasons. Most implementers get annoyed when we require them to implement things that might take significant effort. While Clang and GCC likely won't give an over damn so long as its not C++-modules levels of difficult (and MSVC ignores us until it ships in a real standard), there's hundreds of C compilers and implementers of WILDLY varying quality. Unlike the 4-5 C++ compilers that exist today, C compilers and their implementers are still cranking things out, sometimes as significant pillars of their (local) software economy. Now, while I personally loathe to use things like lines of code as a functional metric for code, it can help us estimate complexity in a very crude, contextless way. Checking in on Clang's Itanium Mangler, it clocks in somewhere on the order of about 7,000 lines of code. Which really doesn't sound so bad,

until chibicc's entire codebase measures somewhere around 7,300 lines of code.

"Double the amount of code in my entire codebase excluding tests for this feature" very much does not pass the smell test of implementability for C. This is also not including, you know, all the rest of the code required for actually implementing the "parse constructors and destructors" bit. Though, thankfully, that part is a lot less work than the name mangler. and I can guarantee that since there's quite literally hundreds of C implementations, many of them will… "have fun". If two or three different ways to mangle `ObjectType::ObjectType(double)` is bad, wait until a couple dozen implementers who have concerns outside of "C++ Compatibility" -- some even with an active bone to pick with C++ -- are handed a gaggle of features that relies on a core mechanic that is entirely unspecified. I am certainly not the smartest person out there,

but I know a goddamn interoperability bloodbath when I see one.



## But... What If Name Mangling Was not a Problem?

This is the other argument I have received a handful of times on both the C mailing list, and in my inbox. It's not a bad argument; after all, the entire above argument hinges on the idea of stealing the syntax from C++ entirely and copying their semantics bit-for-bit. By simply refusing to do it the way C++ does it, does it make the above argument go away? Thusly appears the following suggestion, which boils down to something like the following snippet. However, before we continue, note that this syntax comes partially from an e-mail sent to me. PLEASE, second-to-last person who sent me an e-mail about this and notices the syntax looks similar to what was in the e-mail: I am not trying to make fun of you or the syntax you have shown me, I am just trying to explain as best as I can. With that said:

```cpp
#include <stdlib.h>

struct nya {
	void* data_that_must_be_freed;
};

_Constructor void nya_init(struct nya *nya, int n) {
	nya->data_that_must_be_freed = malloc(n);
}

_Destructor void nya_clear(struct nya *nya) {
	free(nya->data_that_must_be_freed);
}

int main () {
	struct nya n = {30};
	return 0;
}
```

The following uses the `_Constructor` and `_Destructor` tags on function declarations/definitions to associate either the returned type `struct nya` and the destructed type `struct nya *` (a pointer to an already-existing `struct nya` to destroy). The sequence of events, here, is pretty simple too:

1. `n`'s memory is allocated (off of the stack), its memory is taken from the appropriate location on the stack and passed to;
2. `nya_init`, which then calls `malloc` to initialize its data member;
3. the `return 0` is processed, storing the `0` value to do the actual return later, while;
4. `nya_clear` is called on the memory for `n`, and the data member is appropriately `free`d;
5. finally, `main` returns `0`.

It has the same deterministic destruction properties as RAII here. But, notably, it is attached to a free-floating function.

This does the smart thing and gets around the name mangling issue! The person e-mailing me here has sidestepped the whole issue about sharing syntax with C++ and its function overloading issue, which is brilliant! If you can associate a regular, normal function call with these actions, it is literally no longer necessary to provide a name mangling scheme. It does not need to exist, so nobody will implement one: it's just calling a normal function. (Kudos to Rust for figuring part of this out themselves as well, though they still need name mangling thanks to Traits and Generics.) It avoids all of the very weird fixes *other* people tried to propose on the C standards internal mailing list by saying things like "only allow one constructor" or "make C++ have `extern "C"` on constructors work and then have C and C++ mangle them differently" or "just implement name manglers for all C compilers that implement C2y/C3a, it's fine". Implementability can certainly be achieved with this.

Other forms of this come from a derivation of the two existing Operators proposals (Marcus Johnson's [n3201](https://www.open-std.org/jtc1/sc22/wg14/www/docs/n3201.pdf) and Jacob Navia's [n3051](https://www.open-std.org/jtc1/sc22/wg14/www/docs/n3051.pdf)), most particularly n3201. The recommendation for n3201 by the author was to just use a different "association" that did not actually affect the syntax of the function itself, so the above code that produces the same affect but under n3201's guidance (but slightly modified from the way it was presented in n3201 because that syntax has Problems™) might look like:

```cpp
#include <stdlib.h>

struct nya {
	void* data_that_must_be_freed;
};

void nya_init(struct nya *nya, int n) {
	nya->data_that_must_be_freed = malloc(n);
}

void nya_clear(struct nya *nya) {
	free(nya->data_that_must_be_freed);
}

_Operator = nya_init;
_Operator ~ nya_clear;

int main () {
	struct nya n = {30};
	return 0;
}
```

Completely ignoring syntax choices here and the consequences therein, these `_Operator` statements would associate a function call with an action. `=` in this case seems to apply to construction, and `~` seems to apply to destruction. As usual, because the association is made using a statement and type information at compile-time, the compiler can know to simply call `nya_init` and `nya_clear` without needing to set up a complex, implementation-internal name mangling scheme to figure out which constructor/member/whatever function it needs to call to initialize the object correctly. It also doesn't rob C++ of its syntax but try to impose different semantics. Nor does it just tell C implementations the functional equivalent of "git gud" with respect to implementing the name mangler(s) required to play nice with other systems. There is, unfortunately, one really annoying problem with having this form of constructors and destructors, and it's the same problem that C++ had when it first started out trying to tackle the same problem back in the 80s and 90s:

none of these proposals come with an Object Model, and C does not have a real Object Model aside from its Effective Types model!




# RAII: C++ Semantics

While the syntax problem can be designed around with any number of interesting permutations or fixes, whether it's `_Operator` or `_Constructor` or whatever, the actual brass-and-tack semantics that C++ endows on memory obtained from these objects is very strict and direct. When someone allocates some memory and casts it to a type and begins to access it, both [[c.malloc]](https://eel.is/c++draft/c.malloc) and [[intro.object]](https://eel.is/c++draft/intro.object#11)/11-13 cover them by giving them *implicitly created objects*, so long as those types satisfy the requirements of being trivial and implicitly-creatable types. On top of that, for constructors and destructors, there is an ENORMOUSLY robust system that comes with it beyond these implicitly created objects. This post was going to be extremely long, but thanks to an excellent writeup by Basit Ayuntande, [everything anyone needs to know about the C++ object model](https://basit.pro/cpp-object-lifecycle/) is already all written up. To fully understand all the details, shortcuts, tricks, and more, **please** read Basit's article; becoming a better C++ developer (if that's desirable) is an inevitably from digesting it.

This, of course, leaves us to talk about just C and RAII and how those semantics play out.



## C: Effective Types

In C, we do not have a robust object model. The closest are *effective type* rules, and they work VIA lvalue accesses rather than applying immediately on cast. The full wording is found in [§6.5.1 "General" of N3220](https://www.open-std.org/jtc1/sc22/wg14/www/docs/n3220.pdf#subsection.6.5.1), which states:

> The effective type of an object for an access to its stored value is the declared type of the object, if any. If a value is stored into an object having no declared type through an lvalue having a type that is not a non-atomic character type, then the type of the lvalue becomes the effective type of the object for that access and for subsequent accesses that do not modify the stored value. If a value is copied into an object having no declared type using memcpy or memmove, or is copied as an array of character type, then the effective type of the modified object for that access and for subsequent accesses that do not modify the value is the effective type of the object from which the value is copied, if it has one. For all other accesses to an object having no declared type, the effective type of the object is simply the type of the lvalue used for the access.

This is a bunch of text to say something really simple: if a region of memory (like a pointer obtained from `malloc`) is present, and it is cast to a specific type for the purposes of reading or writing, that region is marked with a given type and the type plus region informs what is the *effective type* of the memory. The first write or access is what solidifies it as such. The _effective type_ follows a memory region through `memmove` or `memcpy` done with appropriate objects of the appropriate size. Fairly straightforward, standard stuff. The next paragraph after this then creates a list of scenarios wherein about any accesses or writes performed through casts or pointers aliasing that region afterwards:

> - a type compatible with the effective type of the object,
> - a qualified version of a type compatible with the effective type of the object,
> - the signed or unsigned type compatible with the underlying type of the effective type of the object,
> - the signed or unsigned type compatible with a qualified version of the underlying type of the effective type of the object,
> - an aggregate or union type that includes one of the aforementioned types among its members (including, recursively, a member of a subaggregate or contained union), or
> - a character type.

This is, effectively, C's aliasing rules. Once a type is set into that region of memory, once casting happens from one type to another (e.g. casting it first to `uint32_t*` to write to it, and then try to read it as a `float*` next), that action must be on that list to be standard-sanctioned. If it isn't, then undefined behavior is invoked and programs are free to behave in very strange ways, at the whim of implementations or hardware or whatever. While I am not holding the person who sent me the simple one-off e-mail accountable to this, in the wider C ecosystem and in discussion even on the C mailing list, there seemed to be a distinct lack of appreciation for how thought-through the C++ system is and **why it is this way in the first place**. This also becomes glaringly clear after reading n3201 and going through 95% of the discussions around "RAII in C" that just tries to boil it down to simple syntactical solutions with basic code motion. The bigger picture is NOT being considered. There is not even a tiny amount of respect for where C or C++ comes from. It is not just about effective types and shadowy rules about how do they handle dynamic memory: even simpler things just completely fall apart in these counterproposals. Take, for example, a very simple question.



## "How do you handle copies?"

Taking the `_Operator` example from above again, let's add a single line of spice to this:

```cpp
#include <stdlib.h>

struct nya {
	void* data_that_must_be_freed;
};

void nya_init(struct nya *nya, int n) {
	nya->data_that_must_be_freed = malloc(n);
}

void nya_clear(struct nya *nya) {
	free(nya->data_that_must_be_freed);
}

_Operator = nya_init;
_Operator ~ nya_clear;

int main () {
	struct nya n = {30};
	struct nya n2 = n; // OH SHIT--
	return 0;
}
```

In a proposal like n3201, what happens here? The actual answer is "the proposal literally does not answer this question". Assuming (briefly, if I can be allowed such for a moment) the "basic" or "default" for how it works right now, the answer is probably "just `memcpy` like before", which is **wrong**. n3201 is not the first "just do a quick RAII in C" proposal sent to me over e-mail to make this mistake. Simply performing a memberwise copy of `struct nya` from `n` to `n2` leads to an obvious double-free when `n2` goes out of scope, `free`s the memory pointed to by `data_that_must_be_freed`, and then `n` will attempt attempt to free that data as well. This is an infinitely classic blunder, and in critical enough code becomes a security blunder. The suggestions that stem from pointing this out range from unserious to just disappointing, including things like "just ban copying the structure". Nobody needs a degree in Programming Language Design to communicate that "just ban simple automatic storage duration structure copying" is a terrible usability and horrific ergonomics decision to make, but that's where we are. And it's so confusingly baffling that it is impossible to be mad that the suggestion is brought up.

Or, take in n3201's case (which updates the previous paper, [n3182](https://www.open-std.org/jtc1/sc22/wg14/www/docs/n3182.pdf)). When responding to the (ever-present) criticism that operators -- including for initialization/assignment -- that someone could do something weird inside of the operator, n3201 adds a constraint which reads:

> Functions must contain the matching operator in their function bodies. i.e. `_Operator` declarations that associate the compares-equal operator with a function, must contain the compares-equal operator in the body of the function named in the `_Operator` declaration. (iostream-esque shenanigans with overloading the bitwise shift operators to read/write characters and strings isn’t allowed).

The fact that the proposal has something for initialization (but not cleanup), does not mention anything about the fact that the code snippet in the proposal itself apparently (?) leaks memory, that this constraint is very much **deeply** unsettling to impose on any type (there's plenty of `vec4` or other mathematics code where I'm using intrinsics that look nothing like the operators they're being implemented for) does not seem to bother the author in the slightest. Instead, there's just a palpable hatred of C++ there, apparently so strong that it overrides any practical engagement with the problem space. The proposal -- and much of the counter-backlash I had to sift through on the mailing lists and elsewhere as people proposed stripped down RAII solutions for C under the guise of being "simple" -- is too busy taking potshots at C++ to address clear and present dangers to its own functionality.




# C as an Anti-C++

And this is where things just keep getting worse, because so much of C's culture *seems* to swirl around the idea of either being "budget, simple, understandable C++" or "Anti/Nega-C++". Instead of engaging on C's stated merits or goals, like:

- what-you-write-is-what's-inside (a function `foo` produces a binary symbol named `foo`);
- uncompromised, direct access to the hardware (through close collaboration with implementation-defined `asm`, intrinsics, and unparalleled control of the compiler (severe work in progress, honestly));
- simple enough that it can always be used to glue two or more languages together (for any single given platform/compiler combination);
- and, being a smaller language focused on its use cases (K&R literally sold C on being good at strings -- we can see how that's been going in the last 30 years).

We instead get "why doesn't this PRIMITIVE, UNHOLY C just become C++" proposals, and similar just-as-ill-considered "here is my simpler (AND BETTER THAN THAT CRAPPY C++ STUFF) feature" proposals. Sometimes, like the person who e-mailed me with the `struct nya` example, there's a genuine curiosity for exploring a different design space that serves as an actually better basis. But at even our highest echelons, the constant spectre of C++ that continually drives an underlying and utterly unhelpful antagonism that prevents actual technical evaluation. It results in things like `_Operator` throwing itself in the way of RAII, to try and half-heartedly solve the RAII problem without actually engaging with the sincere, instructive merit of the C++ object model. It also prevents actually evaluating the things that make RAII weak, including [problems with the strong association with objects](https://www.open-std.org/jtc1/sc22/wg14/www/docs/n3199.htm#cpp.compat-constructors.destructors) that actually manifest [in its own standard library](https://www.open-std.org/jtc1/sc22/wg14/www/docs/n3199.htm#cpp.compat-destructor.failure).

The negative tradeoffs for `defer` are numerous, especially since it absolutely loses many of the abilities that come from being a regular object with a well-defined lifetime. This means it is not as powerful as constructors and destructors, including that it is prone to Repeat-Yourself-Syndrome since the `defer` entity itself is not reusable. It cannot be attached to individual members of a structure, nor can it be passed through function calls or stored on the heap. It cannot be transferred with move constructors or duplicated with copy constructors in a natural way, or in any way as a matter of fact! It can only exist at function scope, not at file scope, and only exists procedurally.

The beneficial tradeoffs are it avoids the Static Initialization Order Fiasco that comes with having objects with constructors at file scope or marked `static` at function scope. It also does not combine lambdas with object-based destructors to torch 15+ years of life asking the C++ Standards Committee to standardize `std::scope_guard` only to ultimately be denied success at retirement age (sorry, Peter Sommerlad) because of the C++ Standard Library's ironclad exceptions-and-destructors rule. And, to be clear, it was the right decision for them to do that! Poking a hole in the "all destructors from the standard library are `noexcept`" mandate adds needless library complexity gymnastics for a feature that the language should be taking care of! The proper realization after that would be that a language feature is required to sidestep the concerns that come with the Object Model. Of course, I do not expect the C++ Standard Committee's Evolution Working Group to take that situation seriously as a body; likely, they will leave Library Evolution Working Group out to dry on the matter. 

Coming to these sorts of conclusions only arises through behaving as an engineer that is looking to improve at their craft and strengthen their tools, rather than getting into a hammer-measuring pissing contest with the engineers down the hall.




# But. Alas!

It still leaves a sour taste, though. It sort of lingers at the back of anyone's mouth when they sit down to think about it, because it is kind of distasteful.

Genuinely, I understand that C can be behind. **Very** behind, in fact: taking 30 years to standardize `typeof`, not performing macro-gymnastics to get to `typeof_unqual` in the same 30 years, and not making any meaningful moves to work on things like e.g. "Statement Expressions" (something even the Tiny C Compiler implements) easily illustrates just how gut wrenchingly difficult it is to move the needle just a centimeter in this increasingly Godless industry. But when people propose a feature that has had 40+ years of work and refinement and care put into it, but at no point do they sit down and think about "what happens if I copy this object using the usual syntax" or "do we need some syntax for moving objects from one place to another" or "maybe I should not provoke a double free in the world's most harmless looking code", the thoughts start coming in. *Is* this being taken seriously? Is it just forgetfulness? Is it just so automatic nobody thinks about it? Is the pedagogy what is behind here, and is there a teaching crisis for this language?




# So Many Questions

And yet, I will see not one damn answer, that's for sure. Genuinely, I yearn for it because getting things half-baked things like they are in n3201 or similar is kind of rough to deal with. On one hand there's the overwhelming urge to just grab the proposal and rip it up and get a white board and just go "here, HERE. WHERE IS YOUR OBJECT MODEL. WHAT HAPPENS TO THE EFFECTIVE TYPE RULES. DID YOU THINK ABOUT COPYING AND MOVING THINGS. WHAT HAPPENS IF SOMEBODY USES THESE IN AN COMPOUND ASSIGNMENT EXPRESSION. WHAT HAPPENS IF THEY ARE ASSIGNED FROM A TEMPORARY. HOW DO YOU PASS THAT IN TO THE USER. WHAT ARE THE THINGS THEY CAN CONTROL. HOW DO WE HANDLE THIS FROM HEAP MEMORY OR A STACK ARRAY UNSIGNED CHARACTERS."

But that kind of tone, that sort of engagement is antagonistic, probably in the extreme.

It's also not how I would like to engage with anyone. Like, the person who sent me an e-mail with the cute `struct nya` and the very simple and nice `_Constructor` syntax might not even have gotten that deep in the C standard and likely barely knows the effective type rules; I sure as hell barely understand them and I'm in charge of goddamn editing them when a few of the big upcoming papers finally make their way through the C Committee.

If I respond to an e-mail like that -- with all the capital letters and everything -- it **would** be completely out of line and also would be very unfair, because it is not their fault. I haven't done that to anyone so far, but the fact that the thought exists in my head is Not Fun™. It's not anyone's fault, it's just an internal struggle with thinking the whole industry is a lot farther along on these problems and continuously feeling like I am very much too stupid to be here. Like, I'm a goddamn moron, a genuine idiot, I **cannot** be ahead of the game, am I being pranked? Am I being tested, to see if I really belong here? Is someone going to swing in out of the blue and go "AHA, YOU MISSED THE OBVIOUS!"? Something is absolutely not adding up.

The utterly pervasive and constant feeling that a lot of people -- way too many people -- are really trying to invent these things from first principles and pretend like they were the first people to ever conceive of these ideas... it feels pretty miserable, all things considered. Going through life evaluating effectively no prior art in other languages, domains, C codebases as they exist today, just... anything. It's a constant nagging pull when working on things like standard C that for the life of me I cannot seem to shake no matter how carefully I break it down. Hell, part of writing this post is so I can stick a link to it in my `defer` paper and in the `defer` Technical Specification when it happens so I don't have to sit down and walk through why I chose a procedural-style, object-less idiom for C rather than trying to load the RAII shotgun and point it at our beloved 746-and-counting page C standard.

Changing a programming language's whole object model is hard. Adding "things that must be run in order to bring an object into existence, and thing that must be run in order to kill an object, modulo Effective Type rules, with No Other Exceptions" is a big deal. Where in the proposals do they discuss `new`/`delete`, and why they are used as wrappers around `malloc` to ensure construction and destruction are coupled with memory creation to prevent very common bugs? Where is the consideration for placement new or being able to call destructors manually on an object or a region of memory? RAII enables simple idioms but it is **not** a simple endeavor! Weakening portions of RAII makes it so much less useful and so much less powerful, which is really weird! Is not the thing people keep telling me about C is that its the language of ultimate power and ultimate control? Why does that repeatedly not show up in these discussions?!

It feels so bizarre to have to actually sit down and explain some of these things sometimes because a lot of these things have become second nature to me, but it is just a part of the curse.

[![](/assets/img/2024/05/abyss-expert.png)](https://twitter.com/nickm_tor/status/860234274842324993)




# "It was Just Some E-mails, Man, Calm Down!"

To be very clear, the person who sent the e-mail -- whose syntax I stole using `struct nya *` for this post for the `_Constructor`/`_Destructor` idea -- is not someone I actually expect to send me a 5 page e-mail thesis on enhancements to the C object model. That person CLEARLY was just trying to give me a quick simple idea they thought up of that made it easy on them / solved the problem at hand, and I certainly don't fault them for thinking of it! Their initiative actually demonstrates that rather than just doing the copy-paste roboticism of people who would blindly steal syntax from C++ and then strip off the bits they don't like and go "See? Simple!" they're actually thinking about and engaging with the technical merits of the problem. I certainly wish n3201 and other solutions had a fraction of that spark and curiosity and eagerness to explore the space and actually push the needle for C forward, rather than just being driven by trying to define C as "anti-C++".

My intention is to keep moving forward with proposals like `defer`, among many others over the next few years, to start actually improving C for C's sake. Sometimes this will mean cloning an idea right out of C++ and putting it in C; other times, weighing the pros and cons and addressing the inherent deficiencies in approaches to produce something better will be much more desirable. Knee jerk reactions like those in n3201 rarely serve to help either language and are producing demonstrably worse outcomes; which also concern me because I had an idea for handling operators in C for a long time now and seeing the current proposals do a poor job of handling the landscape is not going to bolster anyone's confidence in how to do it…!

But, the person who inquired VIA e-mail deserves an enthusiastic "NICE", a thumbs up, and maybe a cookie and a warm glass of milk for actually thinking about the problem domain. … In fact.

Cookies and milk sounds real good right now… 💚

{% include anchors.html %}
