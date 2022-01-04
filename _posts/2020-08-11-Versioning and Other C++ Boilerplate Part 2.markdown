---
layout: post
title: Versioning and Boilerplate, Benefits++
permalink: /versioning-and-other-boilerplate-part-2
feature-img: "assets/img/2020-08-11/pexels-suzy-hazelwood.jpg"
thumbnail: "assets/img/2020-08-11/pexels-suzy-hazelwood.jpg"
tags: [C++, C, ⌨️]
excerpt_separator: <!--more-->
---

This is a smaller article to make sure I don't forget a fun extra property I've learned [about the system we discussed last time](/versioning-and-other-boilerplate), dear reader!<!--more-->



# A Quick Recap

Last time, we created a system for macro configuration that allowed us to do 3 important things:

- Stop our own code from `#if`-ing on undefined macros and getting the "default value" of 0, leading to typos and misspelled words never being caught by the preprocessor;
- Add a level of indirection that allowed users to never deal with the internal complexity of supporting the above bullet point; and,
- Making use of the configuration painless and repeatable in the library code.

It looked like this:

```cpp
#define SOL_IS_ON(OP_SYMBOL) ((3 OP_SYMBOL 3) != 0)
#define SOL_IS_OFF(OP_SYMBOL) ((3 OP_SYMBOL 3) == 0)

#define SOL_ON          +
#define SOL_OFF         -
```

It was then deployed to define configuration values, like so:

```cpp
#if defined(SOL_STRINGS_ARE_NUMBERS)
	#if (SOL_STRINGS_ARE_NUMBERS != 0)
		#define SOL_STRINGS_ARE_NEVER_NUMBERS_I_ SOL_OFF
	#else
		#define SOL_STRINGS_ARE_NEVER_NUMBERS_I_ SOL_ON
	#endif
#else
	#define SOL_STRINGS_ARE_NEVER_NUMBERS_I_ SOL_ON
#endif
```

I worked on deploying some of these tactics in sol2, and I found at least 4 places where I fat-fingered the keys or had a crusty, old define that I had never properly cleaned out of the code or fully documented. This has helped immensely improve the quality of configuration in my library, and made me double-check all the configuration macros which I stated were used, but actually were not of the right spelling or similar.



# But There's More!

There is also a little extra benefit, however, to doing the macro checking and versioning in this fashion. For example, take `SOL_STRINGS_ARE_NUMBERS`! It's a user-facing definition that, if detected, turns on some additional type coercion that matches pre-existing Lua behavior. It is, by default, off, because type safety with overloaded functions and other cool sol2 features depend on type checking to work:

```cpp
sol::state lua;

lua.set_function("f", sol::overload(
	[](double num) {
		std::cout << "Got a number: " << num << "\n";
	},
	[](std::string str) {
		std::cout << "Got a string: " << str << "\n";
	}
));

// Or

lua.set_function("f",
	[](std::variant<double, std::string> str_or_number) {
		/* std::visit, etc. etc... */
	}
);
```

If someone was working in Lua and they called this with `f("240")`, expecting the `std::string` branch to be taken, vanilla Lua rules would disappoint them. The `double` branch would be called. Thusly, sol2 imposes a special "hey strings aren't actually numbers this isn't JavaScript y'all lmaoooo" type check.

However, legacy code is real, and some people develop this expectation of Lua, no matter what. Therefore, it is important to allow someone to change this behavior by defining `SOL_STRINGS_ARE_NUMBERS`. Unfortunately, most people don't actually read the "Config and Safety" page of the documentation, come across this behavior in sol2, and file an Issue, send me an e-mail, or pop into the Discord to muse about This Strange Behavior Of The Code.

One way to improve this is to provide a better error message when the type check fails. For example, if someone has this:

```cpp
sol::state lua;

lua.set_function("func", []([[maybe_unused]] double number) {
	/* whatever */
});
```

And they are a first-time sol2 user, they might expect `func("42")` to compile, run, and work without a type error being thrown / printed to the console. Instead, they get an error:

```cpp
//sol2: error occured - argument at index 1 is a 'string',
//      expected 'number'
```

Of course, this is foreign to the user because they naturally expected it to work. (Remember, Lua has this behavior already and sol2 exists in a world where user behaviors and expectations are pre-set.) Therefore, a better error message would be this:

```cpp
// sol2: error occured - argument at index 1 is a 'string',
//      expected 'number' (type 'string' is not automatically
//      treated as a number unless 'SOL_STRINGS_ARE_NUMBERS' is
//      defined).
```

This is a much more clear error, and directs the user on what to do. The only problem is that this error still shows up **even if the user explicitly turns it off**:


```cpp
#define SOL_STRINGS_ARE_NUMBERS 0

/* ... */

sol::state lua;

lua.set_function("func", []([[maybe_unused]] double number) {
	/* whatever */
});

lua.script("func('42')");
// sol2: error occured - argument at index 1 is a 'string',
//      expected 'number' (type 'string' is not automatically
//      treated as a number unless 'SOL_STRINGS_ARE_NUMBERS' is
//      defined).
```

This is problematic. We want the error message to only contain the additional error information if the user has no idea about `SOL_STRINGS_ARE_NUMBERS`. If they turned it on or off, that means they have opted into the behavior and know about it already. Just checking `SOL_IS_ON`/`SOL_IS_OFF` is not enough information to know, due to only having a binary decision. Unfortunately, there's no way to check for a default value in the macro...



# Or Is There?

Turns out, the properties of math are great:


```cpp
#define SOL_IS_ON(OP_SYMBOL) ((3 OP_SYMBOL 3) != 0)
#define SOL_IS_OFF(OP_SYMBOL) ((3 OP_SYMBOL 3) == 0)
#define SOL_IS_DEFAULT_ON(OP_SYMBOL) ((3 OP_SYMBOL 3) == 1)

#define SOL_ON                 +
#define SOL_OFF                -
#define SOL_DEFAULT_ON         /
```

We define `SOL_DEFAULT_ON` to be the division operator. (I was also tempted to call it `SOL_FILE_NOT_FOUND`, but even if it is an "internal" definition users don't use I had to resist.) It works with `SOL_IS_ON` because `3 / 3 == 1`, which is not zero. It works with `SOL_IS_OFF`, because `3 / 3 == 1`, which is still not zero (and thus means it is not turned off). But, by checking for the explicit value of `1`, we know for a fact that the internal definition we created means "it is on, AND it is on by default". If you were so inclined, you could use `*` instead and perform multiplication, checking explicitly for the value of `9`. Either way works, honestly.


This means we update our configuration:

```cpp
#if defined(SOL_STRINGS_ARE_NUMBERS)
	#if (SOL_STRINGS_ARE_NUMBERS != 0)
		#define SOL_STRINGS_ARE_NEVER_NUMBERS_I_ SOL_OFF
	#else
		#define SOL_STRINGS_ARE_NEVER_NUMBERS_I_ SOL_ON
	#endif
#else
	#define SOL_STRINGS_ARE_NEVER_NUMBERS_I_ SOL_DEFAULT_ON
#endif
```

Now we have 3 states, 2 of which overlap. `ON` and `DEFAULT_ON` both mean "On", with a special check one can do to see if it was "left on" because the user didn't do anything. This means that in our last sol2 code example, we now have the power to check if you actually did tell sol2 to shut up:

```cpp
#define SOL_STRINGS_ARE_NUMBERS 0

/* ... */

sol::state lua;

lua.set_function("func", []([[maybe_unused]] double number) {
	/* whatever */
});

lua.script("func('42')");
// sol2: error occured - argument at index 1 is a 'string',
//      expected 'number'.
```

There. We know the user is smart to `SOL_STRINGS_ARE_NUMBERS` now! So, in our code when we define the error message, we just write the following:

```cpp
const char mismatched_argument_str_error[] = "sol2: error occured - argument at index {0}"
	" is a 'number', expected 'string'"
#if SOL_IS_DEFAULT_ON(SOL_STRINGS_ARE_NEVER_NUMBERS_I_)
	"(type 'number' is not automatically"
	"treated as a number unless 'SOL_STRINGS_ARE_NUMBERS' is"
	"defined)."
#endif // Handle strings are never numbers being auto-defined by the configuration
;
```

This allows us to create error messages that guide user to do the right thing, but only if they actually don't give a damn. If they do give a damn, they can save a handful of ROM bytes and never have to see that error message at all.

And that's it! It's a bit heavily invested, but I like the idea that I can improve my user's experience.

And also get them to stop sending me e-mails about this because they never read the safety/configuration page of the documentation and are still running with the library without doing any configuration. 😁

Toodle-oo. 💚


<sub><sub><sub>P.S.: if anyone can figure out how to also do `SOL_IS_DEFAULT_OFF` to work while still keeping the general on/off split to be zero versus non-zero, that'd be pretty dope too!</sub></sub></sub>
