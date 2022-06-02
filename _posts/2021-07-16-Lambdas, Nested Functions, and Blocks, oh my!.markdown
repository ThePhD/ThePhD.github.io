---
layout: post
title: Lambdas, Nested Functions, and Blocks, oh my!
permalink: /lambdas-nested-functions-block-expressions-oh-my
feature-img: "/assets/img/2021/07/nest.jpg"
thumbnail: "/assets/img/2021/07/nest.jpg"
tags: [ABI, C, C++, Functions, Lambdas, Motivation, 📜]
excerpt_separator: <!--more-->
---

I have the fortunate privilege to be part of the ISO C Standard mailing list, and recently a thread kicked off<!--more--> about Lambdas and what their need is in the C Community. That thread was in response to an ongoing push by Jens Gustedt's proposal [N2736](https://www.open-std.org/jtc1/sc22/wg14/www/docs/n2736.pdf), where Gustedt is building steam to put a proper function + data type into the C Standard at some point. What kicked off in that thread was a lot of talking about nested functions, blocks, statement expressions, whether we even need the ability to have data + code in C, and more. Several times I wrote a feature-length film of a response to that mailing list, but since the misconception seems to spread far beyond just the C Committee I decided I would publish my analysis of Lambdas, Nested Functions, and Blocks.

I will do my best to cover each of the solutions in pretty good detail. First, let's start with where we are today, with how we have to do function calls and callbacks with plain, vanilla, Standard C code.




# Plain C

Not too much to say here, except that this is the baseline we'll be "scoring" each of the alternatives against. Using functions and handling associated data with code usually requires a tiny bit of API sculpting, but nothing that's really overtly special since C99. Here's a basic example:

```cpp
int compute_with(int x, int(* user_func)(int)) {
	int completed_work = (x + 2) * 3;
	return user_func(completed_work);
}

int f (int y) {
	return 1 + y;
};

int main () {
	return compute_with(1, f);
}
```

Let's say we compute some extra data in `main`, called `x`. If we want to have extra data to our `user_func` here, we have to add an extra little pizazz to our `compute_with` function. So, we add the "omni" parameter type `void*` into the argument list:

```cpp
typedef int(compute_func_t)(int, void*);

int compute_with(int x, compute_func_t* user_func, void* user_data) {
	int completed_work = (x + 2) * 3;
	return user_func(completed_work, user_data);
}

struct variables {
	int x;
};

int f (int y, void* user_data) {
	struct variables* p_variables = (struct variables*)user_data;
	return p_variables->x + y;
}

int main () {
	int x = 1;
	struct variables vars = {x};
	return compute_with(1, f, &vars);
}
```

This is the most generic form. In the above example, you could cut out the structure entirely and just pass a single pointer to `x`. But, this is normally how these sorts of callbacks are handled. It happens everywhere: epoll, libjansson, POSIX, Win32 API, whatever. The formula is always the same: make a `struct`, copy everything into said `struct`, and fire it off to the function as a pointer. Things get marginally more complicated if you have an asynchronous API, where the lifetime of the operation may outlive what you control directly in `main` or any other function. The solution there is to put things on the heap, with `malloc` or similar:

```cpp
#include <stdlib.h>

typedef int(compute_func_t)(int, void*);
typedef void(compute_func_done_t)(void*);

extern int wait_async_done(void);

extern int async_compute_with(int x,
	compute_func_t* user_func,
	compute_func_done_t* free_func,
	void* user_data);

extern int answer;

struct variables {
	int x;
};

int f (int y, void* user_data) {
	struct variables* p_variables
		= (struct variables*)user_data;
	return p_variables->x + y;
}

int dispatch (void) {
	int x = 1;
	void* data = malloc(sizeof(struct variables));
	if (data == NULL) {
		return 0;
	}
	struct variables vars_init = {x};
	struct variables* p_vars = data;
	*p_vars = vars_init;
	async_compute_with(1, f, &free, p_vars);
	return 1;
}

int main () {
	if (!dispatch())
		return 0;

	wait_async_done();
	return answer;
}
```

Alright. So in the context of these three examples, we have a good idea of what we might be looking for. There are obviously far more complex arrangements and setups that can be performed, but this is about as illustrative of the typical function patterns as we can get. We won't talk about every example for every implemenation extension below, but we will talk about the merits of each design as we go along!

Let's get to it. First up: GCC's Nested Functions.




# Nested Functions

Nested functions are a non-standard GCC extension that is provided in some other C implementations. They also have an extensive history in both Fortran and Ada. Since vanilla, ISO Standard C does not allow you to define a function within a function (only declare it), GCC took the syntactic space:

```cpp
int main () {
	int x = 1;
	int f (int y) {
		// a nested function!
		return x + y;
	};
	return f(1);
}
```

This work has been the centerpiece of a lot of libraries that use function pointers for delegating work that lack `void*` userdata pointers, such as the C Standard Library's own [`qsort` function](https://en.cppreference.com/w/c/algorithm/qsort). GCC has baked in the ability to convert these complicated Nested Functions into function pointers. For example, this works:

```cpp
int compute_with(int x, int(* user_func)(int)) {
	int completed_work = (x + 2) * 3;
	return user_func(completed_work);
}

int main () {
	int x = 1;
	int f (int y) {
		return x + y;
	};
	return compute_with(1, f);
}
```

This has led to entire C libraries taking only function pointers and relying on the GCC extension to, effectively, give them the power they need. As one user writes on Stack Overflow:

> Nested functions can access variables of the function in which they are declared. How do you manage this with non-nested functions?  
> …  
> @MatthewMitchell: the problem is how to pass the function like an argument to another. See [this example](https://github.com/sisoputnfrba/so-commons-library/blob/846af05d3b4a2beb3ac8026f66c1d1a2c73f20c8/tests/unit-tests/test_list.c#L265-269): how can you work around that without nested functions?
>
> — Stack Overflow user [mgarciasaia](https://stackoverflow.com/questions/2929281/are-nested-functions-a-bad-thing-in-gcc#comment26596291_2929348), August 9th, 2013

However, the design choices by GCC when this type was first rolled out has led to some unfortunate issues, ones that usually lead to it being completely banned in most contexts. For example, the [Windows Subsystem for Linux](https://github.com/microsoft/WSL/issues/286) completely bans the use of nested functions indirectly by outlawing what's called an "executable stack".



## Executable wha-?

In the early computing days, in order to make Nested Functions work as function pointers, GCC needed a way to reserve space for calling a function that may ultimately work on variables that were not part of the current stack frame / lexical scope. (A lexical scope in the sense of the C programming language does not correspond 1:1 to the current stack frame or function invocation space, but it's suitable to explain this issue.) When you do what the above example using `compute_with` does and let a function pointer to a nested function "escape" the local scope, you trigger in GCC a mode that has to assume that the Nested Function (`f`, in this case) can be invoked even when the local variable `x` is not part of the current call frame. So, GCC uses a concept known as [trampolining](https://nullprogram.com/blog/2019/11/15/#trampoline-illustration).

Effectively, it creates a little area of stack. It writes the function's code to the local stack and produces a function invocation — let's call it `f_trampoline` — there that will set things up, and **then** call the real `f`. This allows GCC to need to avoid `malloc` or any other dynamic allocation behind your back: it just makes a cozy little extra space of stack and, no matter how deeply nested your runtime call tree is, re-wires the `f` function pointer call to point to `f_trampoline` instead. `f_trampoline` makes sure everything is ready (keeps around a pointer to the stack nearby, etc. etc.), then calls the real, underlying `f` you wrote in `main`. All good so far, except...

GCC turns something called the NX bit (No-eXecute bit) off.



## Absolutely Verboten

An extremely criminal offense in the modern security world, executable stack prevention was one of the cheapest mitigations/hardenings in Computer Security. If a user's programs wrote garbage into, say, a typical `char definitely_a_string_folks[45]` array, an attacker couldn't just craft some malicious input to write code into `definitely_a_string_folks` and then exploit a vulnerability to jump to it and run from there. It would unconditionally fail, because you were not allowed to execute the stack. GCC turning your stack executable - silently! - with trampolines is a little "nyeheheh! 😈" to security-minded people everywhere. It ended up getting a crap reputation, pretty quickly, amongst lots of developers:

> Oh, man! Nested C functions are evil. Just don't do it.
>
> — RE: GCC nested functions? [David Mosberger](http://lkml.iu.edu/hypermail/linux/kernel/0405.1/0978.html), May 12th, 2004

GCC eventually got a warning to tell people when GCC would ~~set them up the bomb~~configure their static/shared libraries and executables to do this in `-Wtrampoline`. But, Security-Enhanced Linux, some BSDs, and several other places basically ban or extremely discourage such constructs from compiling altogether.

There are, of course, other designs. With the advent of threads and separate stack spaces, GCC could just make an entirely separate slice of stack and then mark that executable, with the hopes that even if someone messes up data on any thread currently holding the main data, the trampolines and saved stack frames on the separate stack will be less of a security issue. That doesn't solve WSL's issue of just blanket-banning executable stacks, and it makes security reviewers in codebases need to know the intricacies of your build system and compiler flags to make sure GCC uses this hypothetical new mode. GCC could also employ what is known as function descriptors, a not-new low-level technology for describing a function, what it does, and how it's called which may allow for ways to properly keep executable code off the stack. There is, unfortunately, one gigantic problem with this whole approach, and it's [one I've talked about before](https://thephd.dev/intmax_t-hell-c++-c)!



## A B I

Aha, did you really think C could advertise "Stability at the binary level, forever", and think we'd get away from this three-letter demon just because C is "simple"? ABI stands for Application Binary Interface, and it's real bad for us. Even if GCC flipped the script and said "to hell with trampolines, function descriptors are the new hotness baybeeee let's goooooo", think about the code snippet from before:

```cpp
int compute_with(int x, int(* user_func)(int)) {
	int completed_work = (x + 2) * 3;
	return user_func(completed_work);
}

int main () {
	int x = 1;
	int f (int y) {
		// a nested function!
		return x + y;
	};
	return compute_with(1, f);
}
```

If someone put `compute_with` in a shared object (DLL) on a POSIX-based machine, that convention must last for eternity until the code is recompiled AND all its dependencies are recompiled as well. (Which, if you listen to maintainers and implementers, the answer to that question is "lmao never wtf xD recompile? all the code?? dude on whose infra you doin' all THAT???".) If the way the non-trampoline version of `compute_with` differs in how its function pointer looks and/or is used by the trampoline version, new versions of GCC setting up the non-trampoline code will suffer an ABI break where the way the function pointer is used might not line up properly. Some people did seem to hint at making a change, in response to a vulnerability exploited around executable stack and similar shenanigans with Nested Functions:

> … "Most if not all C++ compilers are able to produce code from lambdas (similar to nested functions) without compromising the call stack." It'd be helpful to explore this more and see whether there's any fundamental difference preventing reuse of the same approach (whatever it is) for nested functions as well. I'd appreciate discussion of this on oss-security. My guess is this probably doesn't fit in the existing ABI for C, but I might be wrong.
>
> — Re: GCC Compiler Induced Vulnerability - affects programs compiled with GCC 7 and GCC 8 containing nested functions, [Solar Designer](https://seclists.org/oss-sec/2018/q4/91)

Unfortunately, Mr. Designer is right. It is an ABI break, because to use function descriptors and similar that are deployed in Ada and other architectures [effectively requires setting a bit that is NEVER set in normal function pointers](https://gcc.gnu.org/onlinedocs/gccint/Trampolines.html). In a world where every single bit of a program shares an entire global process/address space (the typical POSIX model), you can't "section" things off by default. Deploying the function descriptor technique in GCC means snapping user machines into pieces in the most frustrating and brutal-to-track ways, making it impossible to change the implementation strategy for nested functions without a hard, full-recompilation. This is why most ABI breaks are usually reserved for sweeping architecture changes, like transitioning from a typical x86 environment to your own personal embedded chip.

It only gets worse, of course!



## Now what?

Multithreading, of course! If you use a nested function in a multithreaded environment please prepare for plenty of pain: nested functions do not create a "copy" of any variables they access, it accesses them by referring to the existing variable directly. [This program has data races](https://godbolt.org/z/c3c719zca) and is completely illegal. It can be expected that nested functions would have this kind of problem, because multithreading wasn't that big of a deal at the time. They all thought it'd require specialist hacks to do multithreading, and it turns out multithreading is perfectly serviceable on normal computers with normal code and a tiny bit of a care. So... oops, I guess? If you need a copy of data given to a nested function, you need to explicitly make copies and farm them out in some bespoke fashion to each nested function, so… uh.

Good luck. Or something!



## Even so.

Despite these shortcomings, Nested Functions have been in use for a long time. When someone can afford to use it and box themselves into a fairly strict sub-ecosystem within the greater land of C, these can work pretty nicely and is likely the reason why many developers cling to the feature, security concerns and all. Executable stacks and other shenanigans aside, it's not too bad and definitely an improvement over the Standard C code! Clang, of course, looked at the security implications and all that other stuff and said "nah, fam". They don't have nested function pointers, but created something else instead...




# Blocks

It wouldn't be C if there weren't at least 2 completely mutually exclusive implementations of what is effectively the same feature set, right? This is part of the expense of having such a flimsy and weak standard: compiler extensions locking you into a specific vendor are rife in the industry. Clang's little darling sweetheart that nobody can convince it to drop for other things is "Blocks".

Blocks are Clang's (or rather, Objective-C's) take on GCC's nested functions. In illustrating Blocks, we must note it has 2 features. The first is the Block itself that is a sequence of statements to execute, and the second is a new "Block pointer" type that goes with it:

```cpp
#include <Block.h>

int main () {
	int x = 1;
	// "block pointer" declaration
	int (^f)(int) =
		// "block literal" expression
		^ int(int y) {
			return x + y;
		};
	return f(1);
}
```

Blocks, unlike GCC's nested functions, are generally allocated in the place where the compiler can figure it out best. The documentation specifically mentions that "the Block referenced is of opaque data that may reside in automatic (stack) memory, global memory, or heap memory". The reality is that in increasingly complex code, the way this works out is that when using block literals, you MUST be prepared for the worst-case-scenario, which is that someone's done something "heinous" enough with Blocks that the compiler gives up trying to perform the optimization and says "into heap memory you go!". This means that your implementation:

- generally, must have `malloc` (or similar) available to play nice with the feature in its totality; and,
- must explicitly handle them being copied.

Not great, but the next bit makes up for it a little!



## Wide Function Pointers!

Clang's Blocks explicitly avoid the executable stack problems of GCC Nested Functions and prevent more lifetime errors thanks to its reference counting mechanism. Of course, this means that Blocks can't be converted to function pointers, in both their trivial and non-trivial cases:

```cpp
#include <Block.h>

int compute_with(int x, int(* user_func)(int)) {
	int completed_work = (x + 2) * 3;
	return user_func(completed_work);
}

int main () {
	int x = 1;
	int (^f)(int) = ^ int(int y) {
		return x + y;
	};
	int (^f_with_no_inner_variables)(int) = ^ int(int y) {
		return 1 + y;
	};
	//  error: initializing 'int (*)(int)' with an expression of incompatible type 'int (^)(int)'
	int a = compute_with(1, f);
	//  error: initializing 'int (*)(int)' with an expression of incompatible type 'int (^)(int)'
	int b = compute_with(1, f_with_no_inner_variables);
	return a + b;
}
```

Clang could have a trampoline (they even have an [LLVM instruction for it](https://llvm.org/docs/LangRef.html#trampoline-intrinsics)), but they refuse to use it (for good reasons). This is for the better, even if it means that `qsort`-like APIs will suffer. So, you need the `void*` parameter technique:

```cpp
#include <Block.h>

typedef int(compute_func_t)(int, void*);

int compute_with(int x, compute_func_t* user_func, void* user_data) {
	int completed_work = (x + 2) * 3;
	return user_func(completed_work, user_data);
}

int wrap_f(int arg0, void* user_data) {
	int ((^(*f)))(int) = (int (^(*))(int))user_data;
	return (*f)(arg0);
}

int main () {
	int x = 1;
	int (^f)(int) = ^ int(int y) {
		return x + y;
	};
	return compute_with(1, wrap_f, &f);
}
```

This looks pretty good! We deduct some points as we still need a `wrap_f` function, similar to plain C. We can't write it next to the creation of `f`, so it has to be extracted out of the function and set up in some manner (with forward declarations or whatever else is necessary). Thankfully, it does not require the `variables` structure like the Standard C code.

The hand-written wrapper being exported outside the function body is probably the only syntactic / usability failure for this part. Normally it doesn't matter; but, get into a 10,000 line "Core Business Logic" function, and then let's see how well you remember to keep everything together or if you don't mess up the function types (which won't warn, because you're casting through `void*` and all the type safety is gone)! There is, unfortunately, bigger problems to deal with.



## Automatic Duration? Heap Variable? Global memory?

The answer to all these questions: `¯\_(ツ)_/¯`

This is seriously a pain in the ass when it comes to making hard guarantees about where the memory is and if it's safe to access it. What happens to `f` in the following example is hard to say:

```cpp
#include <Block.h>
#include <stdlib.h>

typedef int(compute_func_t)(int, void*);
typedef void(compute_func_done_t)(void*);

extern int wait_async_done(void);

extern int async_compute_with(int x,
	compute_func_t* user_func,
	compute_func_done_t* free_func,
	void* user_data);

extern int answer;

int wrap_f(int arg0, void* user_data) {
	int ((^(*f)))(int) = (int (^(*))(int))user_data;
	return (*f)(arg0);
}

void f_done(void* user_data) {
	// ... nothing?
}

int dispatch (void) {
	int x = 1;
	int (^f)(int) = ^ int(int y) {
		return x + y;
	};
	async_compute_with(1, &wrap_f, &f_done, &f);
	return 1;
}

int main () {
	if (!dispatch())
		return 0;

	wait_async_done();
	return answer;
}
```

What's the lifetime of `f` here? How long does it last? Are we guaranteed it's on the heap? The specification doesn't really say. It gives a lifetime analysis for things that are either milled through `BLock_copy` or milled through the `__block` storage specifier. Normally, the fix for this would be to say "ah, well, let's just do `sizeof(f)` and then `malloc` it, and then call `free`!". Unfortunately, when Clang designed Blocks, they made them reference types. Copies are not value-based copies, but shallow, cheap pointer copies. Doing a manual `malloc` means nothing. Copying it to global storage inside the `malloc`'d memory means nothing. Instead, we need to forcefully elevate this type to the heap using `Block_copy`:

```cpp
#include <Block.h>
#include <stdlib.h>

typedef int(compute_func_t)(int, void*);
typedef void(compute_func_done_t)(void*);

extern int wait_async_done(void);

extern int async_compute_with(int x,
	compute_func_t* user_func,
	compute_func_done_t* free_func,
	void* user_data);

extern int answer;

int wrap_f(int arg0, void* user_data) {
	int ((^(*f)))(int) = (int (^(*))(int))user_data;
	return (*f)(arg0);
}

void f_done(void* user_data) {
	int ((^(*f)))(int) = (int (^(*))(int))user_data;
	Block_release(f);
}

int dispatch (void) {
	int x = 1;
	int (^f)(int) = Block_copy(^ int(int y) {
		return x + y;
	});
	async_compute_with(1, &wrap_f, &f_done, &f);
	return 1;
}

int main () {
	if (!dispatch())
		return 0;

	wait_async_done();
	return answer;
}
```

It's unfortunate that the calls to these things are hidden and there's an extra layer of indirection, but it does (thankfully) give us a way out. It likely would have been far, far better to provide a `Block_location(blk)` intrinsic that could give an enumeration value back telling us where it's allocated so that, at the very least, we could avoid the forced copy. But, as with most secret sauce hidden behind the compiler, and a runtime, user information and control is never part of the picture without [getting your hands dirty with ABI documentation or hacking into the hidden bits](https://twitter.com/_hackbunny/status/1414736429503205377).




## "Don't Pay For What You Don't Use! ... Maybe"

We are, unfortunately, violating the "zero runtime overhead" tradeoff here, even if it provides greater safety in a good chunk of situations. It also has the constraint of trying its best to be compatible with Objective-C, so we don't really get a choice to fix it or improve it much like Nested Functions. Thankfully, at least it makes the right choice in terms of how it handles variables: everything is captured by-value (modulo `__block`-tagged variables that are "elevated" to the status of reference-like holdings). This means if you fired it off to 100 threads, everything works as expected!

All in all, Blocks solve some issues with Nested Functions (memory safety) and some of the boilerplate of Standard C (too-lose, type-unsafe coupling of data + function), but go way too far in too many other directions. It becomes less palatable means of combining code + data by adding additional concerns into the feature and making them inseparable. Stripping the user of so many choices (lack of memory placement, lack of reference counting / copy customization) — while necessary to meet the feature's own laid-out criteria of being safer and likely matching Objective C's ABI — is a net negative.

This brings us to the last choice, that Jens Gustedt brought to the table in his proposal.




# Lambdas

Lambdas are a "newer" take on how to do (anonymous-ish) closures. "Newer" is in quotation marks because it's actually an extremely old practice for many different languages. Lambdas and Closure-like types been around for an incredibly long time in functional and imperative languages like OCaml, Haskell, C#, JavaScript, Java, Lua, and so on and so forth. This post won't get into the history of them in those languages, since our goal is to focus on how they might look in a hypothetical future for C if Gustedt's proposals make headway:

```cpp
int main () {
	int x = 1;
	return [x](int y) {
		int result = x + y;
		return result;
	}(1);
}
```

Your immediate reaction is likely going to be "what the hell?", and I wouldn't blame you. As the proposal explains, lambda expressions using the basic syntax of `[ captures... ] ( args... ) { statements... }` create "complete objects". We "capture" the variable `x` by putting it in the square brackets as above. This is different from both Blocks and Nested Functions, where they "capture" the current surrounding lexical scope (in plain English: "all the current defined and visible variables") by some internal and/or magical means.



## Okay...

![A nurse in scrubs taking off his mask and showing a confused face while asking "... But why?".](/assets/img/2021/07/but-why.jpg)

It's a good question. Why should we need to capture things from the surrounding scope by hand? Isn't just having access to it automatically and letting the compiler "optimize" away unused captures better than nothing? Even Clang's Blocks has that power:

> The compiler is not required to capture a variable if it can prove that no references to the variable will actually be evaluated. Programmers can force a variable to be captured by referencing it…
> 
> — Block Language Specifications: [Block Literal Expressions, LLVM](https://clang.llvm.org/docs/BlockLanguageSpec.html#block-literal-expressions), June 16th, 2021

This is where I have to give an — unfortunately — terrible answer that is nonetheless still valid. You see, many times when we're discussing things amongst the C Committee, one of the blockers people throw up to helpful features of any degree is "but it's harder to implement and requires a lot of effort by compiler authors". If we were to run off and go standardize something like Blocks, what we'd be forced to do is put the above quote from Clang's Block specification in the "Recommended Practice" section of the wording for the feature. That's a non-binding, hint-hint-wink-wink-nudge-nudge at compiler implementers to "do the right thing". Unfortunately, many compiler authors would absolutely give our recommended advice a gigantic middle finger.


### ... Wha?

Now now, I say that like the compiler authors are being vindictive. The reality of the matter is that C has repeatedly and perpetually sold itself as being a "simple" language. And I mean...

Is it?

Can't add 2 numbers together in C without consulting the holy standard about whether or not some UB's been tripped, let alone with a well-defined way to figure out how to stop it. We recently just had to reinforce a Defect Report where we stated that "yes, even if a compiler can figure out that your array bounds are, in fact, a constant number, we have to treat the creation and usage of the array like a VLA because the **Standard's** constant expression parser isn't smart enough!".

C is not a simple language.

That's not what management gets told, of course. What management hears is the spicy dream, the "any **good** developer can bang out a C compiler drunk out of their mind on a weekend". The way the Standard supports that dream is by making all of the good stuff people get used to in GCC, Clang, EDG or whatever else "optional" or "recommended". What's actually guaranteed to you by the C Standard is so pathetically miniscule it's sad (and even that tiny little bit is still complex!). It's why every person who ran off to "write their own C compiler" did a miserable job, why every embedded chipset thought they could roll their own C compilers and ended up with a bug-ridden mess. It's why there are so many `#ifdef __SUN_OS` and `// TODO: workaround, please remove`s that end up becoming permanent fixtures for 17 years.

That gets reflected in our C Standard meetings, discussions, and what we guarantee to you.

If we ask for "magic" powerful enough to guarantee your unreferenced variables don't pollute your Block or Nested Function? Hah! You best believe the magic will be "quality of implementation". On Clang or GCC you flip the `-O2` switch and all of your Nested Block Function Closure-Whatever-The-Hells dissolve into immediate function calls that are beautifully inlined and there's no problems or dynamic allocation or funky usage. You port to some god awful (REDACTED)® spinoff compiler custom-made for this 10 year old extremely custom chip and it always calls `malloc` and dynamically allocates and there's no culling of uncaptured variables and for some reason it's pulling the whole damn stack along for the ride and now your jaw is dropping on the floor and management is sending e-mails!

Where's the product, wasn't this supposed to be an easy port? You backtrack and shuffle around and consult the standard and oh my god it's not required! It's not required they do the smart thing *WHY would they do the bad implementation*, what kind of quality of implementation, is this how could the compiler writers **BETRAY me like this** you shoot off an e-mail and get back a response that this is indeed standards conforming and that's just what the specification says and maybe (MAYBE?!?) in the next (PAID?!) version it'll be better and you feel your teeth clench and you taste the copper in your mouth as the chant rises in your head to reach out to the mailing list and your knuckles whiten while the building Brutal Epithets make your chest push out like you're barely holding in t̲͖͍͔͠h̦̯̼e̼̳͖͙̱̮ ̧G҉̳̬̫̬̜r̷͙͎͈̣̙̦e̮̯̰̺a̼͍̦t̩͕̹̳̪̦̖̀ ͚͔D͈̤̠̕e̤̼͇ͅͅf̴͎̣͕̺̬i̺̱̦͙̲̖l̵̤̠ę͍̥̦̻̜ŕ̤ͅ'̛̲̗̠̥s̬̺͉̯̲ breath which compels you to r̤̖̣͎̱͡a̬̩̺͎n͇͠ṱ̀,͚͓ ̶R̤̹̝A̱̱̤̜͎͢NT̴͉̘ͅ,͎̭̼̪͙̀ **RANT**—

…

Captures should be explicit and Jens Gustedt made the right choice. 🙂



## What's in a Name?

The rest of the syntax is easy to follow: there's an opening parentheses to start the usual argument list, and then an opening brace after the argument list to start the body. As the name "lambda expressions" implies, they are also **expressions**, not declarations or statements. That means they can be used anywhere an expression can be used, which can come in handy for what some folks call "immediately invoked function expressions" (IIFE). The above code is an example of an IIFE, where we get to effectively stick statements arbitrarily within the usual return statement's slot for an expression.

One benefit of this is that, wherever an expression is valid, you now have a means to perform arbitrarily complex one-off things in the form of a lambda! The bad news is that, since this is an expression and not a two-part declaration/definition deal like we usually get with C, we run into a bit of a problem. It becomes a little impossible to determine what, exactly, we should call a variable of lambda type. In order to ease this tension for the rest of this article, we're going to use a GCC extension called `__auto_type` (C++ developers will be familiar with this as the `auto` feature):

```cpp
int main () {
	int x = 1;
	__auto_type f = [x](int y) {
		return x + y;
	};
	return f(1);
}
```

Okay, we can give a name to these lambdas now! But, without this extension, this probably chalks up to being a significant disadvantage to Gustedt Lambdas. They cannot be given names since they are not split into the normal declaration / definition that C has for most entities. If we're going to take such a significant hit to potential usability [without introducing something like `auto`](https://www.open-std.org/jtc1/sc22/wg14/www/docs/n2735.pdf), then what are the potential gains? Well, while generating a completely unique type when evaluated and making a complete object might produce this problem, it also gives us some incredibly interesting (and very familiar!) levels of control!



## Wrapping to... `f`?

Let's take our enhanced `compute_with` example that uses the `void*` parameter. A lambda can be made to work there by capturing the variable and using a similar technique to Clang's Block pointers, BUT! We actually can't write a simple `wrap_f` function, because the lambda type is unique! So the question becomes, how do we make... this, all work?

```cpp
typedef int(compute_func_t)(int, void*);

int compute_with(int x, compute_func_t* user_func, void* user_data) {
	int completed_work = (x + 2) * 3;
	return user_func(completed_work, user_data);
}

int wrap_f(int arg0, void* user_data) {
	???? f = (????)user_data;
	return (*f)(arg0);
}

int main () {
	int x = 1;
	__auto_type f = [x](int y) {
		return x + y;
	};
	return compute_with(1, wrap_f, &f);
}
```

There's `????` in that example marking we don't know what the type is supposed to be. How, exactly, can we extract the value of it here? This is where a very handy feature of Gustedt's Lambdas come into play, with *Function Literals*. Function Literals are a fancy way of talking about a Lambda that has **no captures**. Function Literals can convert into normal C function pointers, because they're not carrying around any associated state. So, they act just like your regular functions! (Have all the same properties, too!) Which means we get to use a cool little trick with `__typeof`:

```cpp
typedef int(compute_func_t)(int, void*);

int compute_with(int x, compute_func_t* user_func, void* user_data) {
	int completed_work = (x + 2) * 3;
	return user_func(completed_work, user_data);
}

int main () {
	int x = 1;
	__auto_type f = [x](int y) {
		return x + y;
	};
	typedef __typeof(f) lambda_t;
	compute_func_t* f_dispatch = [](int y, void* ptr) {
		lambda_t* f_ptr = (lambda_t*)ptr;
		return (*f_ptr)(y);
	};
	return compute_with(1, f_dispatch, &f);
}
```

We've made a function pointer `f_dispatch` now. While `f` itself cannot be converted to a function pointer (a similar deficiency from Clang Blocks), the lambda expression from `f_dispatch` can because it has no captures. No captures means no state, and that means capture-less lambdas can be used in every single situation normal function pointers can be used today. Combining it with the `__typeof` extension, we can form a pointer to the Lambda (remember, it's just a normal "complete object"!) and then call it in our `f_dispatch` function. The code is better than the plain C version, and almost identical to the Clang Blocks version:

- it remains local: there's no need to write a function definition outside the function or forward-declare anything; and,
- it requires no special handling; we don't need to define a "these are my variables" `struct`.

It's still not as compact as the Nested Functions version, but nobody is willing to wade into ABI wars with every implementer on the planet just so they can call an API `qsort`-style. More points should be taken off if we were being strict about `__typeof`, but [there's a paper that's finally going to put it in C properly](https://www.open-std.org/jtc1/sc22/wg14/www/docs/n2724.htm). So, we'll overlook the use of `typeof` and `auto` since there are ongoing and active (leaning successful) efforts to make these things happen.



## Dynamic Memory

Part of Clang Block's trade-offs was basically saying all Blocks had to be dynamically allocated unless the compiler could prove otherwise. We know that while that system works, it makes it pretty difficult to strictly control where function data is stored. We'd have to use `Block_copy` to guarantee copies to the heap or we'd have to avoid touching the Block _at all_ and hope it's in the right section of global memory.

What if we could just... put it where we want it?

```cpp
#include <stdlib.h>

typedef int(compute_func_t)(int, void*);
typedef void(compute_func_done_t)(void*);

extern int wait_async_done(void);

extern int async_compute_with(int x,
	compute_func_t* user_func,
	compute_func_done_t* free_func,
	void* user_data);

extern int answer;

int dispatch (void) {
	int x = 1;
	__auto_type f = [x](int y) {
		return x + y;
	};
	typedef __typeof(f) lambda_t;
	void* data = malloc(sizeof(f));
	if (data == NULL) {
		return 0;
	}
	lambda_t* f_heap = data;
	*f_heap = f;

	compute_func_t* f_dispatch = [](int y, void* ptr) {
		lambda_t* f_ptr = (lambda_t*)ptr;
		return (*f_ptr)(y);
	};
	async_compute_with(1, f_dispatch, &free, data);
	return 1;
}

int main () {
	if (!dispatch())
		return 0;

	wait_async_done();
	return answer;
}
```

I don't have to force a copy with `Block_copy`: we don't need special APIs to do what we normally do. What we're allocating changes (the actual complete object rather than a `struct variables`), but now the two are paired together. I can use `free` directly as the `compute_func_done_t` parameter still, rather than write a wrapper for `Block_release`. And, if I want to, I can swap out `malloc` for any other function I want to without needing to [invade the privates of the runtime](https://stackoverflow.com/questions/64405126/c-blocks-extension-libblocksruntime-use-custom-memory-allocator-boehm-gc-f). I can assign the variable into the allocated space, like I expect to normally do. All of that is normal, standard stuff we're all used to by now.



## Multithreading with Lambdas?

Because Lambdas use explicit capture and capture-by-value by default, there's no reason to worry about creating accidental race conditions. If you want to load the shotgun and aim it at your own foot, create a pointer and capture it directly in a lambda and go nuts:

```cpp
int main () {
	typedef void(func_t)(void);
	
	int x = 0;
	int p_x = &x;
	__auto_type f = [p_x](void) {
		/* it's your leg, homie. */
		*p_x += 4;
	};
	typedef __typeof(f) lambda_t;
	func_t* f_dispatch = [](void) {
		lambda_t* f_ptr = (lambda_t*)ptr;
		return (*f_ptr)();
	};
	super_cool_aync_stuff(f_dispatch, &f);
	/* synchronization and whatever */
	return 0;
}
```

At the very least, it's clear from the `&x` that you're willingly taking the decision to wreck yourself. If someone has to sit down and read your documentation to understand that the variable is being accessed by reference, write a test program to prove that to themselves, or run into a bug where a program that begins to use multiple threads suddenly has race conditions, the defaults were likely picked wrong. Both Clang Blocks and Lambdas avoid this problem altogether.



## Weird Syntax and Required `__auto_type`/`auto`

I have to be honest with you, dear reader: even after all this explanation of what it can do, I am not too happy about the syntax. Clang Blocks at least had some special shenanigans that allow you to elide the argument list or the return type of a Block Literal Expression. Unfortunately, due to C's nature we generally only have 2 choices for syntax. It needs to either by symbol soup (`int(^)(int) { ... }` or `[captures..](args...){ ... }`), or `_Ugly_markup(...)` that tons of people end up complaining about. If you want to have explicit captures and not run afoul of C's already fairly wild declarator and expression syntax, finding a better symbol soup for Lambdas or using `_Lambda(captures...) (args...) { ... }` would likely not yield much fruit. You also have to be concerned about C's downstream dependents who consume the language directly. Whether they share headers or write integrated FFI parsers on top of it, you have to be careful about what syntax those might've stolen from you, too.

So, maybe the syntax isn't so bad after all?

Gustedt Lambdas also do not come with a built-in wide function pointer type that "erases" the special, generated type for the lambda. It could be taken care of as a follow-on proposal that takes more things into consideration than Clang's Block Pointers do. There's 2 different compiler extensions that may exist for this: the Clang Block Pointers, and the `__closure` storage specifier used in some more (ancient?) C compilers. While it is nice that Function Literals are a thing, being able to store a "wide function pointer" and break it down into it's constituent parts would make working with lambdas easier. The consequence of that is we need to answer lifetime questions with respects to:

```cpp
int a = 0, b = 1, c = 2;
/* stuff */
_Magic_wide_pointer(int(int)) f = [a, b, c](int d) { /* ... */ return 0; }
```

Does the wide function pointer type keep the lambda expression alive? Do we do what we did for Compound Literals and say the lifetime of the lambda is for the entire "block scope"? Do we only apply it specifically to this scope? It's not *immediately* clear how such a construct would work in the face of complete, statically-sized objects and the lifetime of a normal expression in C. This does not mean the proposal is bad: at some point we should probably come to grips with the fact that under any other design, we would need type erasure or other special tricks to store arbitrarily-sized closures in a singly-typed object. That means that we'd have to leave it to an implementation to make tradeoffs on our behalf, and if both GCC Nested Functions and Clang Blocks have taught us anything it is that compiler writers and/or runtime library authors are not well-equipped to make that tradeoff on behalf of their users in long-term scenarios.

In general, in fact, nobody is qualified to make long-term investments in what tradeoffs should be made in these domains. Which is exactly why...




# It Should Be the Developer's Choice

There is nothing wrong with filling in the blanks with some decent defaults, but more often than not these details become a fundamental part of our ecosystem that then become impossible to change and prevent nice features or forward movement in our ecosystems.

Every time we standardize something that does some weird magic in the background that the developer, personally, cannot interact with (`va_arg`s and the stack, default function argument promotion, default integer promotion when doing math, constant expressions being "implemented defined" but not allowed to show in the front-end, variable length arrays, `char` being signed and used for UTF-8 resulting in negative numbers during integer promotion, and so on) we regret it IMMENSELY. Every major feature we added in C11 we dialed off with C17 to make it optional. Just two weeks ago one of the people teaching about integer promotions got it wrong in spectacular fashion, and debate ensued about the exact rules. The people disagreeing with them weren't even sure if they were disagreeing correctly and everyone was confused for a moment, because that's how **crap** our current situation is.

C's brand of "magic" — especially the kind implementers have swung around in the past — is far more insidious and dangerous than any amount of language complexity and far more poisonous to the ecosystem.

Much of it is antiquated, outdated, and most frustratingly doesn't just serve as an "ah, okay" surprise, but a malicious and often-times *exploitable* bug. I don't want Clang's wizardry with Blocks, I don't want to depend on GCC being smart enough to not mark the stack executable in this one particular case where it can prove the variable never escapes its scope and is therefore never modified and can therefore the nested function can be used like a normal function pointer. I want predicable rules I can give to somebody that hold **all the time**, that have nothing to do with somebody's optimizer.

Lambdas have rules. They apply everywhere. They don't depend on hidden `malloc`s. I don't need [a very impressive](https://twitter.com/ilyakurdyukov/status/1415211514630447106) but nonetheless horrifying foray into assembly to call them. I don't need to figure out how I am going to specify "the stack" or "the current lexical scope, but as a variable" in the C Standard. If I don't need a variable, I toss it out of my `[` capture list `]`. Heck, because I'm writing the capture list explicitly, the compiler can warn me when I'm not using something I captured and I can remove it at my leisure to get some object space back. The object size and binary properties are mine to modify as I see fit. I can make my own "trampolines" and I don't need an executable stack, and the security hole that comes with it. I don't need to probe an ISA pointer to figure out if the implementation decided, on Apple architecture, to put my Block in the right place or not so I can save myself the `Block_copy`/`Block_release`. If I `malloc` my Lambda, I just `free` it. I don't need to worry about GCC's forever-ABI guarantees. I can test against these rules.

I can ship code on multiple platforms and not rewrite platform tests to check how broken somebody's constant folding implementation is.



## No More Compiler Guesswork

Compiler authors can know, without performing complex escape analysis, *exactly what goes into the Lambda by the time they get to the first `{`*. There is no hard thinking, no waiting to read the rest of the function and hoping someone doesn't form a pointer to the Nested Function or Block or whatever. There's no reason somebody has to come to a C Standards Meeting and I have to hear — AGAIN — that something is too hard for a tiny embedded developer or the drunk 40-summat year old to implement for their chip in a weekend. You can implement the ENTIRETY of lambdas as a full rewrite operation in the front end and change nothing about your backend, because Lambdas don't give the implementation room to mess around. Even an **interpreter** implementation can make the same guarantees with the same level of power. All the same optimizations apply, no questions asked! It's a complete object. Not some special function-with-stack-pointer-amalgamation, not some maybe-stack-maybe-heap hybrid monster, not something locked into ABI forever where I'll have to get into trench warfare with ABI stakeholders who'll hold the entire Committee at implementation-gunpoint to kill the feature if things don't jive with decisions some dude made at his desk 26 years ago.

Please. You cannot demand in one breath that everything is recommended practice and unspecified/undefined on one hand, and then complain that users are unhappy with non-standard and hard to port in the very next breath. It does not make sense. If this is how we do business, then acknowledge that and start standardizing explicit opt-in. Stop sprinkling implicit magic on things. Stop claiming C is close to the metal and leaving out "but only with an aggressive compiler that very much goes far beyond the core required standardese". Stop ignoring implementers and security professionals who have literally run the gauntlet before you, only to have a "Come to Jesus" moment after we have piles of CVEs and implementation divergence issues on our desk.

Please. Somebody did the work. Literally, somebody already did the work for Nested Functions and Blocks and Wide Function Pointers and all of it, somebody sat down and designed it and deployed it and we have a wealth of information at our fingertips. We can pinpoint precisely the places they failed, and design not to do that anymore, that's literally what we're here to do.




# Please.

Clang already explored the magic Block space. GCC already explored (and, thanks to ABI, ruined) the nested function space. I understand it's an attractive extension. I understand it has these nice properties. But please consider the rest of us, who have to work on multiple platforms, who are caught between your ABI wars, who are stuck probing your ISA pointer to "save some cycles" and push performance metrics. We do not want special rules, we do not want flashy, we do not want hidden. We don't care if it's ugly. Somebody already did all the hard work, and the space has already been thoroughly explored. I don't want to try to standardize something that people have already taken the time to confirm does not and will not scale to the whole ecosystem, that makes unacceptable tradeoffs.

Please... The need has been made clear after decades of users asking for and investigating the problems. People have been trying to mash function-with-associated-data together for ages. Every API in the past 25 years learned from the utter mistake that was `qsort`. They provide a `void*` parameter and everyone hooks into that to make their various APIs work out. Lua C, io_uring, POSIX, XML parsers, libjpeg, libpng, the list literally cannot be contained about people who take `void*` parameters to make their APIs do amazing things. But it's not type-safe. It's prone to misuse. It's difficult to integrate with other languages. It's verbose and a burden. Listen to your users and to the implementation elders that literally did the work for 2+ decades.

Please? Jens Gustedt may have forgotten to write good motivation in their proposal and left this analysis out. They may have left out all this information. But it is out there. Listen to the security reports and the NX-off-bit-is-banned OS loaders. Listen to the performance people and the other folks trying to get work done on the day-to-day. Listen to the developers filing bugs and trying to get *work* done. Whatever you standardize, whatever you build, make sure we can control it, make sure it's got tractable rules,

and make sure we can get our work done on more than one platform-compiler combination, like a Standard should let us.

Please. 💚


<sub><sub><sub><sub><sub>... Pretty, pretty please... I'm so tired......</sub></sub></sub></sub></sub>

{% include anchors.html %}
