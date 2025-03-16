---
layout: post
title: "The Defer Technical Specification: It Is Time"
permalink: /c2y-the-defer-technical-specification-its-time-go-go-go
feature-img: "/assets/img/2025/03/clock.jpg"
thumbnail: "/assets/img/2025/03/clock.jpg"
tags: [C, C standard, defer, cleanup, safety, finally, 📜]
excerpt_separator: <!--more-->
---

After the Graz, Austria February 2025 WG14 Meeting, I am now confident in the final status of the defer TS, and it is now time.<!--more-->




# ... Time to What?

Time for me to write this blog post and prepare everyone for the implementation blitz that needs to happen to make `defer` a success for the C programming language. If you're smart and hip like Navi who wrote the GCC patch, the maintainer of slimcc who implemented defer from the early spec and found it both easy and helpful, and several others who are super cool and great, you can skip to the [(DRAFT) ISO/DIS 25755 - defer Technical Specification](/_vendor/future_cxx/technical%20specification/C%20-%20defer/C%20-%20defer%20Technical%20Specification.pdf) and get started! But, for everyone else... 




# What is `defer`?

For the big brain 10,000 meter view, `defer` ⸺ and the forthcoming TS 25755 ⸺ is a *general-purpose* block/scope-based "undo" mechanism that allows you to ensure that no matter what happens a set of behavior (statements) are run. While there are many, many more usages beyond what will be discussed in this article, `defer` is generally used to cover these cases:

- `unlock()` of a mutex or other synchronization primitive after a `lock()`;
- `free()` of memory after a `malloc()`;
- `deref()` of a reference-counted parameter after a `ref()` or (shallow) `copy()` operation;
- `rollback` on a transaction if something bad happens;

and so, so much more. For C++ people who are going "wait a second, this sounds like destructors!", just go ahead and [skip down below](#but-what-about-c) and read about the C++ part while ignoring all the stuff in-between about `defer` and WG14 and voting and consensus and blah blah blah.

For everyone else, we're going to go over some pretty simple examples of `defer`, using a series of `printf`'s to construct (or fail to construct) a phrase, just to get an idea of how it works. Here's a basic example showing off some of its core properties:

```cpp
#include <stdio.h>

int main () {
	const char* s = "this is not going to appear because it's going to be reassigned";
	defer printf(" bark!\"");
	defer printf("%s", s);
	defer {
		defer printf(" woof");
		printf(" says");
	}
	printf("\"dog");
	s = " woof";
	return 0;
}
```

The output of this program is as follows:

```sh
$> ./a.out
"dog says woof woof bark!"
```

The following principles become evident:

- The contents of a `defer` are run at the **end** of the block that contains it.
	- `defer` can be nested.
	- The rules for nested `defer` are the same as normal ones: it executes at the end of its containing block (`defer` introduces its own block.)
- Multiple `defer` statements run in reverse lexicographic order.
- `defer` does not need any braces for simple expression statements, same as `for`, `while`, `if`, etc. constructs.
- `defer` can have braces to stack multiple statements inside of it, same as `for`, `while`, `if`, etc. constructs.
- `defer` uses the value of the variable at the time `defer` is run at the end of the scope, not at the time when the `defer` statement is encountered.

This forms the core of the `defer` feature, and the basis by which we can build, compare, and evaluate this new feature.



# "Build?" Wait... Are You Just Making This Up Entirely From Scratch?

Thankfully, no. This is something that has been cooked up for a long time by existing implementations in a variety of ways, such as:

- `__attribute__((cleanup(func))) void* some_var;`, where `func` takes the address of `some_var` and gets invoked when `some_var`'s lifetime ends/the scope is finished (Clang, GCC, TCC, and SO many more compilers);
- `__try`/`__finally`, where the `__finally` block is invoked on the exit/finish of the `__try` block (MSVC);
- and, various different library hacks, such as [this high-quality defer library](https://gitlab.inria.fr/gustedt/defer) and this other [library-based library hack](https://github.com/moon-chilled/Defer).

It has a lot of work and understanding behind it, and a ton of existing practice. Variations of it exist in Apple's MacOS SDK, the C parts of Swift, the Linux Kernel, GTK's `g_autoptr` (and qemu's `Lockable`), and so much more. This, of course, begs the question: if this has so much existing implementations in various styles, and so many years of experience, why is this going into a Technical Specification (or just "TS") rather than directly into the C standard? Well, honestly, there's 2 reasons.

The first reason is that vendors claim they can put it into C ⸺ and make it globally available ⸺ faster than if it's put in the C working draft. Personally, I'm not sure I believe the vendors here; there are many features they have put into C, or even back ported from later versions of C into older versions of C. But, I'm not really at a point in my life that I feel like arguing with the vendors about a boring reskin of feature that's been in C compilers for just under as long as I've been alive, so I'm just going to take their word for it.

The second, more unfortunate, reason is that `defer` was proposed before I got my hands on it. It was not in a good shape and ready for standardization, and the ideas about what `defer` should be were somewhat all over the place. Which is fair, because many of the initial papers were exploratory: the problem was that when we had to cut a C23 release, there was a (minor) panic about new features and there was a lot of concentrated effort to try and slim `defer` down into something ready to go. Going from the wishy-washy status of before that wasn't grounded in existing practice to something material caused the Committee to reject the idea, and state that if it came back it should come back as a TS.

I could argue that this is not fair, because that vote was based off older version of the paper that was not ready and was subject to C23 pressures. The older papers were discussing various ideas like whether to capture variables by value at the point of the `defer` statement (catastrophic) or whether `defer` should be stapled to a higher scope / function scope like Go (also catastrophic), and whether writing a `for` loop would accurate a (potentially infinite) amount of extra space and allocations to store variables and other data that would be needed to run at the end of the scope (yikes!). None of those shenanigans apply anymore, but we still have to go to a TS, even though it's a mirror-image of how existing practice works (in fact, *less* powerful than existing practice). Somewhat recently, we took new polls about whether it should go in a TS or whether it should go directly into the IS (International Standard; the working draft basically). There was support and consensus for both, but *more* consensus for a TS.

It's not really worth fighting about, though, so into a `defer` TS it goes.

My only worry is that Microsoft is going to do what it usually does and ignore literally everybody else doing things and not do any forward progress with just a `defer` TS. (As they do with most GNU or Clang or not-Microsoft extensions, some Technical Reports, and some TSs.) So, the only place we'll get experience is in places that already rely pretty heavily on the existence of the compiler feature. But, I'm more than willing to be pleasantly surprised. It could be driven by users demanding Microsoft make some of their C stuff safer through their User Voice / Feature Request submission portal. But, the message from Microsoft since Time Immemorial was always "just write C++", so I can imagine we'll just get the same messaging here, too, and have to wait until `defer` hits the C Standard before they implement it.

Nevertheless, this TS will be interesting for me. I have several other ideas that should go through a TS process; if I get to watch over the next couple of years that vendors weren't being honest about how quickly they could implement `defer` in their compilers ⸺ if *only* they had a TS to justify it! ⸺ that will strongly color my opinion on whether or not **any** future improvements should use the TS process at all.

So we'll see! In the meantime, however, let's talk about how `defer` differs from its similarly-named predecessors in other languages.



## Scope-based

The central idea behind `defer` is that, unlike its Go counterpart, `defer` in C is lexically bound, or "translation-time" only, or "statically scoped". What that means is that `defer` runs unconditionally at the end of the block or the scope it is bound to based on its lexical position in the order of the program. This gives it well-defined, deterministic behavior that requires no extra storage, no control flow tracking, no clever optimizations to reduce memory footprint, and no additional compiler infrastructure beyond what would normally be the case for typical variable automatic storage duration (i.e., normal-ass variable) lifetime tracking. Here's a tiny example using `mtx_t`:


```cpp
#include <threads.h>

extern int do_sync_work(int id, mtx_t* m);

int main () {
	mtx_t m = {};
	if (mtx_init(&m, mtx_plain) != thrd_success) {
		return 1;
	}
	// we have successful initialization: destroy this when we're done
	defer mtx_destroy(&m);

	for (int i = 0; i < 12; ++i) {
		if (mtx_lock(&m) != thrd_success) {
			// return exits both the loop and the main() function,
			// defer block called:
			// - mtx_destroy
			return 1;
		}
		// now that we have succesfully init & locked,
		// make sure unlock is called whenever we leave
		defer mtx_unlock(&m);

		// …
		// do a bunch of stuff!
		// …
		if (do_sync_work(i, &m) == 0) {
			// something went wrong: get out of there!
			// return exits both the loop and the main() function,
			// defer blocks called:
			// - mtx_unlock
			// - mtx_destroy
			return 1;
		}
		
		// re-does the loop, and thus:
		// defer block called:
		// - mtx_unlock
	}

	// defer block called:
	// - mtx_destroy
	return 0;
}
```

The key takeaway from the comment annotations in the above is that: no matter if you early `return` from the 6th iteration of the for loop, or you bail early because of an error code sometime after the loop:

- if needed, `mtx_unlock` is always called on `m`, first;
- and, `mtx_destroy` is called on `m`, last.

Notably, the `mtx_unlock` call only happens if execution is still inside of the `for` loop, and only happens with exits from that specific scope after `defer` is passed. This is an important distinction from Go, where every `defer` is actually "lifted" from its current context and attached to run at the end of the *function itself that is around it*. This tends to make sense as a "last minute check before a function exits about some error conditions", but it has some devastating consequences for simple code. Take, for example, the following code from above, slightly simplified and modified to make a normal-looking Go program:

```go
package main

import (
	"fmt"
	"sync"
)

var x  = 0

func work(wg *sync.WaitGroup, m *sync.Mutex) {
	defer wg.Done()	
	for i := 0; i < 42; i++ {
		m.Lock()
		defer m.Unlock()
		x = x + 1
	}
}


func main() {
	var w sync.WaitGroup
	var m sync.Mutex
	for i := 0; i < 20; i++ {
		w.Add(1)
		go work(&w, &m)
	}
	w.Wait()
	fmt.Println("final value of x", x)
}
```

The output of this program, [on Godbolt, is](https://godbolt.org/z/9KM1Pe9jE):

```sh
Killed - processing time exceeded
Program terminated with signal: SIGKILL
Compiler returned: 143
```

Yeah, that's right: it never finishes running. This is because this code **deadlocks**: the `defer` call is hoisted to the **outside** of the `for` loop in `func work`. This means that it calls `m.Lock()`, does the increment, loops around, and then attempts to call `m.Lock()` again. This is a classic deadlock situation, and one that hits most people often enough in Go that they have to add a little caveat. "Use an immediately invoked function to clamp the `defer`'s reach" is one of those quick caveats:

```go
package main

import (
	"fmt"
	"sync"
)

var x  = 0

func work(wg *sync.WaitGroup, m *sync.Mutex) {
	defer wg.Done()	
	for i := 0; i < 42; i++ {
		func() {
			m.Lock()
			defer m.Unlock()
			x = x + 1
		}()
	}
}


func main() {
	var w sync.WaitGroup
	var m sync.Mutex
	for i := 0; i < 20; i++ {
		w.Add(1)
		go work(&w, &m)
	}
	w.Wait()
	fmt.Println("final value of x", x)
}
```

This [runs without locking up Godbolt's resource until a SIGKILL](https://godbolt.org/z/e5nrxhb9n). Of course, this is pathological behavior; while it works great for a simple, direct use case ("catch errors and act on them"), it unfortunately results in other problematic behaviors. This is why the version in the defer TS does not cleave strongly to the scope of the function definition (or immediately invoked lambda), but instead directly to the innermost block and its associated scope.

This is not necessarily perfect, but it does mean that `defer` does not need to "store" its executions up until the end of the function, nor does it need to record predicates or track branches to know which `defer` is taken by the end of some arbitrary outer scope.  In fact, for any `defer` block, the model of behavior for the `defer` TS is pretty much that it takes all the code inside of the `defer` block and dumps it out onto each and every translation-time (compile-time) exit of that scope. This applies to early `return`, `break`ing/`continue`ing out of a loop scope, and also `goto`ing towards a label.


## Oh, even `goto`?

In general, `goto` is banned from jumping over a `defer` or jumping into the sequence of statements in a `defer`. It can jump back before a `defer` in that scope. The same goes for trying to use `switch`, `break`/`continue` (with or without a label), and other things. Here's a few examples where things would not compile if you tried it:

```cpp
#include <stdlib.h>

int main () {
	void* p = malloc(1);
	switch (1) {
		defer free(p); // No.
	default:
		defer free(p); // fine
		break;
	}
	return 0;
}
```

```cpp
int main () {
	switch (1) {
	default:
		defer {
			break; // No.
		}
	}
	for (;;) {
		defer {
			break; // No.
		}
	}
	for (;;) {
		defer {
			continue; // No.
		}
	}
	return 0;
}
```

It's also important to be aware that `defer` that are not reached in terms of execution do not affect the things that come before them. That is, this is a leak still:

```cpp
#include <stdlib.h>

int main () {
	void* p = malloc(1);
	return 0; // scope is exited here, `defer` is unreachable
	defer free(p); // p is leaked!!
}
```

Similar to the bans on `break`, `goto`, `continue`, and similar, `return` also can't exit a `defer` block:

```cpp
int main () {
	defer { return 24; } // No.
	return 5;
}
```

Though, if you're an avid user of both `__attribute__((cleanup(...)))` and `__try`/`__finally`, you'll find that some of these restrictions are actually harsher than what is allowed by the mirrored existing practice, today.



## Wait.... Existing Practice Can Do WHAT, Now?

The bans written about in the preceding section are a bit of a departure from existing practice. Both `__attribute__((cleanup(...)))` and `__try`/`__finally` ⸺ the original versions of this present in GCC/Clang/tcc/etc., and MSVC, respectively ⸺ allowed for some (cursed) uses of `goto`, pre-empting `return`s, and more in those implementation-specific kinds of `defer`.

An [MSVC example (with Godbolt)](https://godbolt.org/z/s5vevEbnh):

```cpp
int main () {
	__try {
		return 1;
	}
	__finally {
		return 5;
	}
	// main returns 5 ⸺ can stack this infinitely
}
```

A [GCC example (with Godbolt)](https://godbolt.org/z/G94rEjco8):

```cpp
#include <stdio.h>
#include <stdlib.h>

int main () {
	__label__ loop_endlessly_and_crash;
	loop_endlessly_and_crash:;
	void horrible_crimes(void* pp) {
		void* p = *(void**)pp;
		printf("before goto...\n");
		goto loop_endlessly_and_crash; // this program never exits successfully or frees memory
		printf("after goto...\n");
		printf("deallocating...\n");
		free(p);
	}
	[[gnu::cleanup(horrible_crimes)]] void* p = malloc(1);
	printf("allocated...\n");
	printf("before label...\n");
	printf("after label...\n");
	return 0;
}
```

The vast majority of people ⸺ both inside and outside of the Committee ⸺ agreed that allowing this directly in `defer` for the first go-around was Bad and Evil. I also personally agree that I don't like it, though I would actually be okay with relaxing the constraint in the future because even if I don't personally like what I'm seeing from this, I can still write out a tangible, understandable, well-defined behavior for "`goto` leaves a `defer` block" or "`return` is called from within a `defer` block". The things I won't move on, though, are "`goto` into a `defer` block" (which exit of the scope is the `goto` taking execution to??), or jumping over a `defer` statement in a given scope: there's no clear, unambiguous, well-defined behavior for that, and it only gets worse with additional control flow.

But, even if you can't `return` from the TS's deferred block, you still have to be aware of when and how the `defer` actually runs in relation to the actual expression contained in a `return` statement or similar scope escape.



## `defer` Timing

Matching existing practice and also C++ destructors, `defer` is run before the function actually returns but *after* the computation of the return's value. In a language like this, this is not observable in simple programs. But, in complex programs, this *absolutely* matters. For example, consider the following code:

```cpp
#include <stddef.h>

extern int important_func_needs_buffer(size_t sz, void* p);
extern int* get_important_buffer(int* p_err, size_t* p_size, int val);
extern void drop_important_buffer(int val, size_t size);

int f (int val) {
	int err = 0;
	size_t size = 0;
	int* p = get_important_buffer(&err, &size, val);
	if (p == nullptr || err != 0) {
		return err;
	}
	defer {
		drop_important_buffer(val, size);
	}
	return important_func_needs_buffer(sizeof(*p) * size, p);
}

int main () {
	if (f(42) == 0) {
		printf("bro definitely cooked. peak.");
		return 0;
	}
	printf("what was bro cooking???");
	return 1;
}
```

There's 2 times in which you can run the `defer` block and its `drop_important_buffer(…)` call.

- before the function returns and before `important_func_needs_buffer(…)`;
- or, before the function returns but after `important_func_needs_buffer(…)`.

The problem becomes immediately apparent, here: if the `defer` runs **before** the expression in the `return` statement (before `important_func_needs_buffer(…)`), then you actually drop the buffer before the function has a chance to use it. That's a one-way ticket to a use-after-free, or other extremely security-negative shenanigans. So, the only logical and plausible choice is to run the second option, which is that the `defer` block runs after the `return` expression is evaluated but before we leave the function itself.

This does frustrate some people, who want to use `defer` as a last-minute "`return` value change" like so:

```cpp
int main (int argc, char* argv[]) {
	int val = 0;
	int* p_val = &val;
	defer {
		if ((argc % 2) == 0) {
			*p_val = 30;
		}
	}
	return val; // returns 0, not 30, even e.g. if argc == 2
}
```

But I value much more highly compatibility with existing practice (both `__try`/`__finally` and `__attribute__((cleanup(…))))`), compatibility with C++ destructors, and avoiding the absolute security nightmare. If someone wants to evaluate the `return` expression but still modify the value, they can write a paper or submit feedback to implementations that they want `defer { if (whatever) { return ...; } }` to be a thing. That way, such a behavior is formalized. And, again, even if I don't personally want to write code like this or see code like this, there's still a detectable, tangible, completely well-defined behavior for what happens if a `return` is evaluated in a `defer`. This is also not nearly as complex as e.g. Go's `defer`, because the `defer` TS uses a translation-time scoped `defer`.

It won't result in "dynamically-determined and executed `defer` causes spooky action at a distance". One would still need to be careful about having nested `defer`s that also overwrite the return, or subsequent `defer`s that attempt to change the `return` value. (One would also have to contend that every `defer`-nested `return` would need to have its expression evaluated, and potentially discarded, sans optimization to stop it.) Given needing to answer all of these questions, though, it is still icky and I'm glad we don't have to go through with `return` (or `goto` or `break` or `continue`) within `defer` statements.



## ... What About Control Flow Outside of Compilation Time?

Run-time style control flow like `longjmp`, or similar `_Noreturn`/`[[_Noreturn]]`/`[[noreturn]]`-marked functions, are a-okay if they mimic the above allowed uses of `goto`. If it jumps out of the function entirely, or jumps into a previous scope but beyond the point where a `defer` would be, the behavior can end up undefined. That means use of functions like `exit`, `quick_exit`, or similar explicitly by the user may leak resources by not executing any currently open `defer` blocks. This is similar to C++, where calling any of the C standard library exit functions (and, specifically, NOT `std::terminate()`) means destructors will not get run. The only function that this is not fully true on is `thrd_exit`, as glibc has built-in behavior where `thrd_exit` will actually provoke unwinding of thread resources by calling destructors on that thread. (You can then use `thrd_exit` on the main thread, even in a single-threaded program, as a means to trigger unwinding; this is an implementation detail of glibc, though, and most other C standard libraries don't behave like this.)

The exact wording in the TS and the proposal is that its "unspecified" behavior, but it doesn't actually proscribe any specific set of behaviors that can happen. So, even if we use the "magic" word of "unspecified" for these run-time jumps, the behavior is *effectively* as bad as undefined behavior because there really isn't any document-provided guarantee about what happens when you run off somewhere with e.g. `setjmp`/`longjmp` in these situations. I guess the only thing it prevents is some compiler optimization junkie trying to optimize based on whether or not `defer` with a run-time jump would trigger undefined behavior, though it's effectively an optimization you can maybe get by only combining `defer` and one of these run-time jumps. At that point, I'd question what the hell the engineer was doing submitting that kind of "improvement" in the first place to the optimizer, and reject it on the grounds of "Please find something better to do".

But, you never know I guess?

Maybe there would be real gains, but I'm not holding my breath nor making any space for it. But beyond just ignoring dubious weird optimization corners for `defer`...




# Does.... Defer Actually Solve Any Problems, Though?

Believe it or not: yes. I'm not one to waste my time on things with absolutely no real value; there's just too little time and standardization takes too much damn effort to focus on worthless things[^he-lies]. Though, if you were to take it from others, you'd hear about how `defer` complicates the language for [not much/no benefit](https://www.yodaiken.com/2023/12/04/dont-defer/):

> … The proposal authors show a complex solution to make the code free storage and then show how it can be “simplified” using defer. But it is trivial to centralize cleanup in one function, no new features needed. If I was developing this code for real, I’d take the next step and make it single exit. …
> 
> ⸺ Victor Yodaiken, ["Don't Defer", December 12, 2023](https://www.yodaiken.com/2023/12/04/dont-defer/)

The code Yodaiken is referring to is code contained in the original proposal (the original proposal is being updated in lock-step with the TS), specifically [this section](/_vendor/future_cxx/papers/C%20-%20Improved%20__attribute__((cleanup))%20Through%20defer.html#design.safety). The code in question was offered to me by its author, and I was told to simply / work with the code. So, after a bit of cleanup and checking and review, this is the first-effort `defer` version of the [original code](https://github.com/mortie/housecat/blob/6a1b76a8b41c5ad0fea2698f8559f171feb43c72/src/build/plugins.c):

```cpp
h_err* h_build_plugins(const char* rootdir, h_build_outfiles outfiles, const h_conf* conf)
{
	char* pluginsdir = h_util_path_join(rootdir, H_FILE_PLUGINS);
	if (pluginsdir == NULL)
		return h_err_create(H_ERR_ALLOC, NULL);
	defer free(pluginsdir);
	char* outpluginsdirphp = h_util_path_join(
		rootdir,
		H_FILE_OUTPUT "/" H_FILE_OUT_META "/" H_FILE_OUT_PHP
	);
	if (outpluginsdirphp == NULL)
	{
		return h_err_create(H_ERR_ALLOC, NULL);
	}
	defer free(outpluginsdirphp);
	char* outpluginsdirmisc = h_util_path_join(
		rootdir,
		H_FILE_OUTPUT "/" H_FILE_OUT_META "/" H_FILE_OUT_MISC
	);
	if (outpluginsdirmisc == NULL)
	{
		return h_err_create(H_ERR_ALLOC, NULL);
	}
	defer free(outpluginsdirmisc);
	//Check status of rootdir/plugins, returning if it doesn't exist
	{
		int err = h_util_file_err(pluginsdir);
		if (err == ENOENT)
		{
			return NULL;
		}
		if (err && err != EEXIST)
		{
			return h_err_from_errno(err, pluginsdir);
		}
	}

	//Create dirs if they don't exist
	if (mkdir(outpluginsdirphp, 0777) == -1 && errno != EEXIST) {
		return h_err_from_errno(errno, outpluginsdirphp);
	}
	if (mkdir(outpluginsdirmisc, 0777) == -1 && errno != EEXIST) {
		return h_err_from_errno(errno, outpluginsdirmisc);
	}

	//Loop through plugins, building them
	struct dirent** namelist;
	int n = scandir(pluginsdir, &namelist, NULL, alphasort);
	if (n == -1)
	{
		return h_err_from_errno(errno, namelist);
	}
	defer {
		for (int i = 0; i < n; ++i)
		{
			free(namelist[i]);
		}
		free(namelist);
	}
	for (int i = 0; i < n; ++i)
	{
		struct dirent* ent = namelist[i];
		if (ent->d_name[0] == '.')
		{
			continue;
		}
		char* dirpath = h_util_path_join(pluginsdir, ent->d_name);
		if (dirpath == NULL)
		{
			return h_err_create(H_ERR_ALLOC, NULL);
		}
		defer free(dirpath);
		char* outdirphp = h_util_path_join(outpluginsdirphp, ent->d_name);
		if (outdirphp == NULL)
		{
			return h_err_create(H_ERR_ALLOC, NULL);
		}
		defer free(outdirphp);
		char* outdirmisc = h_util_path_join(outpluginsdirmisc, ent->d_name);
		if (outdirmisc == NULL)
		{
			return h_err_create(H_ERR_ALLOC, NULL);
		}
		defer free(outdirmisc);

		h_err* err;
		err = build_plugin(dirpath, outdirphp, outdirmisc, outfiles, conf);
		if (err)
		{
			return err;
		}
	}
		
	return NULL;
}
```

This code has some improvements over the original, insofar that it actually protects against a few leaks that were happening in that general purpose code. Instead of this approach, Yodaiken instead changed it to this:

```cpp
struct plugins {
	char *pluginsdir;
	char *outpluginsdirphp;
	char *outpluginsdirmisc;
	char *dirpath;
	char *outdirphp;
	char *outdirmisc;
	int n;
	struct dirent **namelist;
};

void freeall(struct plugins *x)
{
	if (x->pluginsdir)
		free(x->pluginsdir);
	if (x->outpluginsdirphp)
		free(x->outpluginsdirphp);
	if (x->outpluginsdirmisc)
		free(x->outpluginsdirmisc);
	if (x->dirpath)
		free(x->dirpath);
	if (x->outdirphp)
		free(x->outdirphp);
	if (x->outdirmisc)
		free(x->outdirmisc);
	for (int i = 0; i < x->n; i++) {
		free(x->namelist[i]);
	}
}

h_err *h_build_plugins(const char *rootdir, h_build_outfiles outfiles,
		       const h_conf * conf)
{
	struct plugins x = { 0, };
	x.pluginsdir = h_util_path_join(rootdir, H_FILE_PLUGINS);
	if (pluginsdir == NULL)
		return h_err_create(H_ERR_ALLOC, NULL);
	x.outpluginsdirphp = h_util_path_join(rootdir,
					      H_FILE_OUTPUT "/" H_FILE_OUT_META
					      "/" H_FILE_OUT_PHP);
	if (outpluginsdirphp == NULL) {
		freeall(&x);
		return h_err_create(H_ERR_ALLOC, NULL);
	}
	x.outpluginsdirmisc = h_util_path_join(rootdir,
					       H_FILE_OUTPUT "/" H_FILE_OUT_META
					       "/" H_FILE_OUT_MISC);
	if (x.outpluginsdirmisc == NULL) {
		freeall(&x);
		return h_err_create(H_ERR_ALLOC, NULL);
	}
	//Check status of rootdir/plugins, returning if it doesn’t exist
	{
		int err = h_util_file_err(x.pluginsdir);
		if (err == ENOENT) {
			freeall(&x);
			return NULL;
		}
		if (err && err != EEXIST) {
			freeall(&x);
			return h_err_from_errno(err, x.pluginsdir);
		}
	}

	//Create dirs if they don’t exist
	if (mkdir(x.outpluginsdirphp, 0777) == -1 && errno != EEXIST) {
		freeall(&x);
		return h_err_from_errno(errno, x.outpluginsdirphp);
	}
	if (mkdir(outpluginsdirmisc, 0777) == -1 && errno != EEXIST) {
		freeall(&x);
		return h_err_from_errno(errno, outpluginsdirmisc);
	}
	//Loop through plugins, building them
	x.n = scandir(x.pluginsdir, &x.namelist, NULL, alphasort);
	if (n == -1) {
		freeall(&x);
		return h_err_from_errno(errno, x.namelist);
	}
	for (int i = 0; i < n; ++i) {
		struct dirent *ent = namelist[i];
		if (ent->d_name[0] == '.') {
			continue;
		}
		x.dirpath = h_util_path_join(x.pluginsdir, ent->d_name);
		if (dirpath == NULL) {
			freeall(&x);
			return h_err_create(H_ERR_ALLOC, NULL);
		}
		x.outdirphp = h_util_path_join(outpluginsdirphp, ent->d_name);
		if (x.outdirphp == NULL) {
			freeall(&x);
			return h_err_create(H_ERR_ALLOC, NULL);
		}
		x.outdirmisc =
		    h_util_path_join(x.outpluginsdirmisc, ent->d_name);
		if (x.outdirmisc == NULL) {
			freeall(&x);
			return h_err_create(H_ERR_ALLOC, NULL);
		}

		h_err *err;
		err =
		    build_plugin(dirpath, outdirphp, outdirmisc, outfiles,
				 conf);
		if (err) {
			freeall(&x);
			return err;
		}
	}

	freeall(&x);
	return NULL;
}
```

This works too, and one would argue that Yodaiken has done the same as `defer` but without the new feature or a TS or any shenanigans. But there's a critical part of Yodaiken's argument where his premise falls apart in the example code provided: refactoring. While he states that in "serious" code he would change this to be a single exit, the example code provided is just one that replaces all of the `defer` or manual `free`s of the original to instead be `freeall`. This was not unanticipated by the proposal he linked to, which not only discusses `defer` in terms of code savings, but also in terms of **vulnerability prevention**. And it is exactly that which Yodaiken has fallen into, much like his peers and predecessors who work on large software like the Linux Kernel.


## CVE-2021-3744, and the Truth About Programmers

The problem, that Yodaiken misses in his example code rewrite and his advice to developers, is the same one that the programmers [responsible for CVE-2021-3744](https://nvd.nist.gov/vuln/detail/CVE-2021-3744). You see, much like Yodaiken's rewrite of the code, the function in question here had an object. That object's name was `tag`. And just like Yodaiken's rewrite, it had a function call like `freeall` that was meant to be called at the exit point of the function: `ccp_dm_free`. The problem, of course, is that along one specific error path, in conjunction with other flow control issues, the V5 CCP's `tag` structure was not being properly freed. That's a leak of (potentially sensitive) information; thankfully, at most it could provoke a Denial of Service, per the original reporter's claims.

This is the exact pitfall that Yodaiken's own code is subject to.

It's not that there isn't a way, in code as plain as C90, to write a function that frees everything. The problem is that in any sufficiently complex system, even with one that has as many eyeballs as bits of the cryptography code in the Linux Kernel, one might not be able to trace all the through-lines for any specifically used data. The function in question for CVE-2023-3744 had exactly what Yodaiken wanted: a single exit point after doing preliminary returns for precondition/invalid checks, `goto` to a series of laddered cleanup statements for the very end, highly reviewed code, and being developed in as real a context as it gets (the Linux Kernel). But, it still didn't work out.

Thankfully, this CVE is only a 5.5 -- denial of service, maybe a bit of information leakage -- but it's not the first screwup of this sort. This is only one of hundreds of CVEs that follow the same premise, that have been unearthed over the last 25-summat years[^spitballing] of vulnerability tracking. And, most importantly, Yodaiken's code can be changed in the face of `defer`, in a way that both reduces the number of lines written and does all the same things Yodaiken's code does, but with better future proofing and less potential leaks:

```cpp
struct plugins {
	char *pluginsdir;
	char *outpluginsdirphp;
	char *outpluginsdirmisc;
	char *dirpath;
	char *outdirphp;
	char *outdirmisc;
	int n;
	struct dirent **namelist;
};

void freeall(struct plugins *x)
{
	if (x->pluginsdir)
		free(x->pluginsdir);
	if (x->outpluginsdirphp)
		free(x->outpluginsdirphp);
	if (x->outpluginsdirmisc)
		free(x->outpluginsdirmisc);
	if (x->dirpath)
		free(x->dirpath);
	if (x->outdirphp)
		free(x->outdirphp);
	if (x->outdirmisc)
		free(x->outdirmisc);
	for (int i = 0; i < x->n; i++) {
		free(x->namelist[i]);
	}
}

h_err *h_build_plugins(const char *rootdir, h_build_outfiles outfiles,
		       const h_conf * conf)
{
	struct plugins x = { 0, };
	defer freeall(&x);
	x.pluginsdir = h_util_path_join(rootdir, H_FILE_PLUGINS);
	if (pluginsdir == NULL)
		return h_err_create(H_ERR_ALLOC, NULL);
	x.outpluginsdirphp = h_util_path_join(rootdir,
					      H_FILE_OUTPUT "/" H_FILE_OUT_META
					      "/" H_FILE_OUT_PHP);
	if (outpluginsdirphp == NULL) {
		return h_err_create(H_ERR_ALLOC, NULL);
	}
	x.outpluginsdirmisc = h_util_path_join(rootdir,
					       H_FILE_OUTPUT "/" H_FILE_OUT_META
					       "/" H_FILE_OUT_MISC);
	if (x.outpluginsdirmisc == NULL) {
		return h_err_create(H_ERR_ALLOC, NULL);
	}
	//Check status of rootdir/plugins, returning if it doesn’t exist
	{
		int err = h_util_file_err(x.pluginsdir);
		if (err == ENOENT) {
			return NULL;
		}
		if (err && err != EEXIST) {
			return h_err_from_errno(err, x.pluginsdir);
		}
	}

	//Create dirs if they don’t exist
	if (mkdir(x.outpluginsdirphp, 0777) == -1 && errno != EEXIST) {
		return h_err_from_errno(errno, x.outpluginsdirphp);
	}
	if (mkdir(outpluginsdirmisc, 0777) == -1 && errno != EEXIST) {
		return h_err_from_errno(errno, outpluginsdirmisc);
	}
	//Loop through plugins, building them
	x.n = scandir(x.pluginsdir, &x.namelist, NULL, alphasort);
	if (n == -1) {
		return h_err_from_errno(errno, x.namelist);
	}
	for (int i = 0; i < n; ++i) {
		struct dirent *ent = namelist[i];
		if (ent->d_name[0] == '.') {
			continue;
		}
		x.dirpath = h_util_path_join(x.pluginsdir, ent->d_name);
		if (dirpath == NULL) {
			return h_err_create(H_ERR_ALLOC, NULL);
		}
		x.outdirphp = h_util_path_join(outpluginsdirphp, ent->d_name);
		if (x.outdirphp == NULL) {
			return h_err_create(H_ERR_ALLOC, NULL);
		}
		x.outdirmisc =
		    h_util_path_join(x.outpluginsdirmisc, ent->d_name);
		if (x.outdirmisc == NULL) {
			return h_err_create(H_ERR_ALLOC, NULL);
		}

		h_err *err;
		err =
		    build_plugin(dirpath, outdirphp, outdirmisc, outfiles,
				 conf);
		if (err) {
			return err;
		}
	}

	return NULL;
}
```

As you can see here, we made one ⸺ just one ⸺ change to Yodaiken's code here: we use `defer freeall(&x)` at the very start of the function and delete it everywhere else. With `defer`, we no longer need to add a `freeall(&x)` at every exit point, nor do we need a ladder of `goto`s cleaning up specific things (in the case where the structure didn't exist and we tried to use a single exit point).

It's not that Yodaiken's change wasn't an improvement over the existing code, it's just that it simply failed to capture the point of the use of `defer`: no matter how you exit from this function now (save by using [runtime control flow](#-what-about-control-flow-outside-of-compilation-time)), there **is** no way to forget to free anything. Nor is there any way to forget to free anything on some specific path. The problems of CVE-2021-3744 ⸺ and the hundreds of CVEs like it ⸺ are not really a plausible issue anymore. It means that the C code you write becomes resistant to problems with later changes or refactors: adding additional checks and exits (as we did compared to the original code in the repository, to cover some cases not covered by the original) means a forgotten `freeall(&x)` doesn't result in a leak.




# This is the power of `defer` in C

Focusing on things that are actually difficult and worth your time is what your talents and efforts are made for. Menial tasks like "did I forget to free this thing or `goto` the correct cleanup target" are a waste of your time. Even the [Linux Kernel is embracing these ideas](https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/tree/include/linux/cleanup.h), because bugs around forgetting to `unlock()` something or forgetting to `free` something are awful wastes of everyone's life, from people who have to report 'n' confirm basic resource failures to getting annoying security advisories over fairly mundane failures. We have more interesting code and greater performance gains to be putting our elbow grease into that do not include fiddling with the same basic crud thousands of times.

This is what the `defer` TS is supposed to bring for C.




# But... What About C++?

For C++ people, MOST (but not all) of `defer` is covered by destructors (and constructors) and by C++'s [object model](/just-put-raii-in-c-bro-please-bro-just-one-more-destructor-bro-cmon-im-good-for-it). The chance of having `defer` in C++, properly, is less than 0. The authors of C++'s library version of this (`scope_guard`) have intentionally and deliberately abandoned having this in the C++ standard library, and efforts to revive it (including efforts to revive it to spite `defer` and tell C to stop using `defer`) have either gone eerily/swiftly quiet or been abandoned. This does not mean there is no dislike or dissent for `defer`, just that its C++ compatriots have seemed to ⸺ mostly ⸺ calm down and step back from just trying to put raw RAII into C. Not that I would fully object to actually working out an object model and having real RAII, as stated in a [previous article](/just-put-raii-in-c-bro-please-bro-just-one-more-destructor-bro-cmon-im-good-for-it) and [in the rationale of the proposal itself discussing C++ compatibility of `defer`](/_vendor/future_cxx/papers/C%20-%20Improved%20__attribute__((cleanup))%20Through%20defer.html#cpp.compat), certainly not! It's just that everyone who's trying has so far done a rather half-baked job of attempting it, mostly in service of their favorite pet feature rather than as a full, intentional integration of a complete object model that C++ is still working out the extreme edge-case kinks of to this day through [Core Working Group issues](https://open-std.org/JTC1/SC22/WG21/docs/cwg_active.html).

There are also some edge cases where `defer` is actually better than C++, as mentioned in the rationale of the proposal. For example, exceptions butt up against the very strict `noexcept` rule for destructors (especially since its not just a rule, but **required** for standard library objects). This means that using RAII to model `defer` becomes painful when you intentionally want to use `defer` ⸺ or `scope_guard` ⸺ as an exception-detection mechanism and a transactional rollback feature. Destructors overwhelming purpose are, furthermore, to make repeatable resource cleanup easy, but in tying it to the object model must store all of the context that is accessible within the object itself so it can be appropriately accessed. Carrying that context can be antithetical to the goals of the given algorithm or procedure, meaning that a lot more effort goes into effective state management and transfer when just having key `defer` blocks in certain in-line cases would save on both object size and context move/transfer implementation effort. One can get fairly close by having a `defer_t<...>` templated type in C++ with all move/copy/etc. functions

Destructors can also fall apart in certain specific cases, like in the input and output file streams of C++. Because the destructor needs to finish to completion, cannot throw (per the Standard Library ironclad blanket rules), and must not block or stall (usually), the specification for the C++ standard streams will swallow up any failures to flush the stream when it goes out of scope and the destructor is run. This usually isn't a problem, but I've had to sit in presentations in real life during my C++ Meetup where the engineers gave talks on standard streams (and many of their boost counterparts) making it impossible for them to have high-reliability file operations. They had to build up their own from scratch instead. (I don't think  Niall Douglass's (ned13's) Low-Level File IO had made it into Boost by then.)

Nevertheless, while RAII covers the overwhelming majority of use cases (reusable resource and policy), `defer` stands by itself as something uniquely helpful for the way that C operates. And, in particular, it can help cover real vulnerabilities that happen in C code due to the simple fact that most people are human beings.

Thusly...




# The Time is Now

[This is the specification for the `defer` TS](/_vendor/future_cxx/technical%20specification/C%20-%20defer/C%20-%20defer%20Technical%20Specification.pdf). If you are reading this and you are a compiler vendor, beloved patch writer, or even just a compiler hobbyist, the time to implement this is **today**. Right now. The whole point of a TS ⸺ and the reason I was forced by previous decisions and discussion out of my control to pick a TS ⸺ is to obtain deployment experience. Early implementers have already found, recovered, and discovered bugs in their code thanks to `defer`. There is a wealth of places where using `defer` will drastically improve the quality of code. Removing a significant chunk of human error as well as reducing risk during refactors or rewrites because someone might forget to add a `goto CLEANUP;` or a necessary `freeThat()` call are tangible, real benefits we can do to prevent classes of leaks.

Implement `defer`. Tell me about it. Tell others about it.

The time is now, before C2Y ships. That's why it's a TS. Whether you gate it behind `-fdefer-ts`/`-fexperimental-defer-ts`, or you simply make it part of the base offering without needing extra flags, now is the time. The Committee is starting to constrict and retract heavily from the improvements in C23, and vendors are starting to get skittish again. They want to see serious groundswells in support; you cannot just sit around quietly, hoping that vendors "get the memo" to make fixes or pick up on your frustrations in mailing lists. Go to them. Register on their bug trackers (and look for existing open bugs). E-mail their lists (but search for threads already addressing things). You must be vocal. You must be loud. You must be direct.




# You Must Not Be Ignorable.

With: compiler vendors ⸺ especially the big ones ⸺ getting more and more serious about telling people to *Do It In The Standard Or #$&^! Off* (with some exceptions); pressure being applied to have greater and greater consensus in the standard itself making that bar higher and higher; and, vendors and individuals getting more and more pissed off about changes to C jeopardizing their implementation efforts and what they view as the integrity of the C language, extensions and changes are more at risk now than ever. Please. Please, please, prettiest of pleases.

Don't let good changes go down quietly. 💚


- Banner and Title Photo by [Ethan Sarkar, from Pexels](https://www.pexels.com/photo/big-ben-tower-illuminated-at-dusk-in-london-31019924/)


### Footnotes

[^spitballing]: Just spitballing the time, I haven't actually checked.
[^he-lies]: Author's note: This is a lie. `#embed` took 7 years total.

{% include anchors.html %}
