---
layout: post
title: sol3: Feature Complete
permalink: /sol3-feature-complete
feature-img: "assets/img/pexels/correct-diverse-diversity.jpg"
thumbnail: "assets/img/pexels/correct-diverse-diversity.jpg"
tags: [C++, sol3, sol2, completed, ⌨️, 🚌]
excerpt_separator: <!--more-->
---

It's finally happening. After the C++Now and CppCon 2018 talks and much work and feedback (thanks to everyone who sent e-mails and all the Patrons and Discord members who participated), sol3 is now<!--more--> feature complete.


# What Does That Mean?

sol3 is a mostly-compatible take on the previous sol2. It means that I will not be working on sol2 anymore (sol2 has already been feature-complete for nearly a year now, and has received mostly bugfixes for). All critical changes are going towards sol3. The biggest changes:

- There is a shiny new Customization Point space for users, with the explicit promise that as a library author I will never write anything that can potentially compete with your Customization Points. sol2 users will need to move their customization points to this new layer (which is fine: the fix is literally deleting a bunch of lines of code and no longer needing to write them in the sol2 namespace).
- There are huge performance gains now for all people who relied on the previously dicey inheritance for usertypes. It is as fast as I could possibly make it without reflection, platform-specific hacks and assembly tricks. Once the Reflection finishes going through PD TS balloting and is approved for C++20/23 and compilers roll it out, I can implement this seamlessly with sol3's new mechanisms without backwards compatibility breaks and gain the last drops of performance that will allow users to reach SWIG levels of inheritance performance without any additional markup.
- Container usertypes have better handling and defaults in some select cases, and slightly better performance. We also fixed an older class of bugs that were nonetheless breaking changes that I could not touch in sol2: it's all clean now.
- CMake integration is now Modern Top Tier™. You can grab sol2, the single, or the generated single now. You get sol2's target by doing `sol2::sol2`, `sol2::single` after `add_subdirectory(sol/single)`, or `sol2::single_generated` after `add_subdirectory(sol/single)`. We also vastly improved header hygiene and similar with new compilation tests, and added a whole new suite of runtime tests. The default for including sol2 is also now `#include <sol/sol.hpp>`, following the `boost::` style of inclusion to help prevent clashes.
- C++17 enabled. The defines enabling C++17 features are there still out of pure laziness: it's a C++17 library now.
- Performance is Good™.

Because this is a major version increase, some of the code people wrote before will break. Here's the good news: _most_ of your code won't break, and nothing too fundamental or core about sol2 actually broke. Just the things that were already terrible. 


# Usertype Removals and Cleanup

I cleaned up a lot here.

- `new_simple_usertype` and the `simple_usertype` objects do not exist anymore. They were hacks to increase compile times while forcing the user to pay runtime costs in certain cases. No longer necessary, and the code is about as fast as it can be at all times now: this has been **removed**.
- `sol::usertype<T>`'s "I take a million things things and create several layers of variadic template crap and bring the compiler to its knees with a 12 GB AST OooOooh!" constructor is dead. `sol::usertype` is now an actual meta-table type that behaves like a regular `sol::table`, where you can set/get things. The wonky constructor with `set_usertype` no longer exists. As it shouldn't: I regret ever writing it.

All code that is wrong produces a build break now, since I prefer that I break your build rather than let you upgrade and then silently do the wrong thing. The cool thing about the breakages here is that it's not just breakages without respite: everything that's broken now has better ways of doing it that compiles faster (though I don't promise you won't end up with a 12 GB heap, especially if you're careless).

One thing I did not break was `lua.new_usertype<blah>( ctor, "key", value, "key2", sol::property(...), /* LITERALLY EVERYTHING */ );`. That will keep working. And it will also bring your compiler to its knees again, which isn't very nice of you. Please consider using 

```cpp
struct blah {
	int a;
	bool b;
	char c;
	double d;
	/* ... */
};

constexr int some_static_class_value = 58;

int main () {
	sol::state lua;

	sol::usertype<blah> metatable = lua.new_usertype<blah>(
		/* just some constructor and destructor stuff maybe*/
	);

	metatable["a"] = blah::a;
	metatable["b"] = sol::property(
		[](blah& obj) { return obj.b; }, 
		[](blah& obj, bool val) { obj.b = val; if (obj.b) { ++obj.a; } }
	);
	metatable["super_c"] = sol::var(some_static_class_value);
}
```

The runtime speed of the resulting binding is the same, thanks to some serious optimizations I perform on making sure we serialize what is as close to an optimized map of `std::unique_ptr<char[]>` -> `callable or variable` I could possibly produce. Usertypes also can handle non-string keys now as well, allowing someone to index into them with userdata (pointers and such), integers, and other data points without needing to override the `__index` or `__newindex` methods.

The amount of improvements is actually staggering, really...! I could write a whole article on the deterministic runtime qualities we've imbued into sol2 now thanks to doing things like having an `unordered_map` that no longer needs to allocate to lookup for keys without having to resort to `boost::` or hacking up our own `unordered_map`. This means that support for people who were using sol2 for real-time processing purposes remains in place, making sure they have a 0-allocation runtime path they can take advantage of for usertypes and similar.


# New Customization Points: Your Own Backyard

Customization points were also poor in sol2. Sometimes, users wanted to change how fundamental types were handled by the system: they could not without writing over my code. I speak for a good moment about why struct specialization points as a fundamental abstraction are bad for a library like sol2 in my [CppCon 2018 video](https://youtu.be/ejvzoifkgAI?t=2243) and how I have to struggle to keep a clean separation.

- `sol::stack::pusher`, `sol::stack::checker`, etc. -- fighting with me to make sure I don't step on your customization points specializations really sucks.
- Specializing those old names now does not compile. The older names no longer exist, to make sure it was a hard compiler error and that the changes I made did not silently change your program's runtime and give you a day's worth of a debugging adventure.

 The performance hasn't changed, but the sloppy code sure did. This hunk of boilerplate-filled code is now replaceable:

```cpp
struct two_things {
	int a;
	bool b;
};

namespace sol {

	// First, the expected size
	// Specialization of a struct
	template <>
	struct lua_size<two_things> : std::integral_constant<int, 2> {};

	// Then, specialize the type
	// this makes sure Sol can return it properly
	template <>
	struct lua_type_of<two_things> : std::integral_constant<sol::type, sol::type::poly> {};

	// Now, specialize various stack structures
	namespace stack {

		template <>
		struct checker<two_things> {
			template <typename Handler>
			static bool check(lua_State* L, int index, Handler&& handler, record& tracking) {
				int absolute_index = lua_absindex(L, index);
				bool success = stack::check<int>(L, absolute_index, handler)
					&& stack::check<bool>(L, absolute_index + 1, handler);
				tracking.use(2);
				return success;
			}
		};

		template <>
		struct getter<two_things> {
			static two_things get(lua_State* L, int index, record& tracking) {
				int absolute_index = lua_absindex(L, index);
				int a = stack::get<int>(L, absolute_index);
				bool b = stack::get<bool>(L, absolute_index + 1);
				tracking.use(2);
				return two_things{ a, b };
			}
		};

		template <>
		struct pusher<two_things> {
			static int push(lua_State* L, const two_things& things) {
				int amount = stack::push(L, things.a);
				amount += stack::push(L, things.b);
				return amount;
			}
		};

	}
}
```

With this much shorter and much more wonderful snippet:

```cpp
struct two_things {
	int a;
	bool b;
};

template <typename Handler>
bool check(sol::types<two_things>, lua_State* L, int index, 
Handler&& handler, sol::stack::record& tracking) {
	int absolute_index = lua_absindex(L, index);
	bool success = sol::stack::check<int>(L, absolute_index, handler) 
		&& sol::stack::check<bool>(L, absolute_index + 1, handler);
	tracking.use(2);
	return success;
}

two_things get(sol::types<two_things>, lua_State* L, int index, 
sol::stack::record& tracking) {
	int absolute_index = lua_absindex(L, index);
	int a = sol::stack::get<int>(L, absolute_index);
	bool b = sol::stack::get<bool>(L, absolute_index + 1);
	tracking.use(2);
	return two_things{ a, b };
}

int push(sol::types<two_things>, lua_State* L, const two_things& things) {
	int amount = sol::stack::push(L, things.a);
	amount += sol::stack::push(L, things.b);
	return amount;
}
```

With the guarantee that any future SFINAE and handlers I write will always be "backend" and "default": if you write one in the new paradigm, it will always override my behavior. You can even override my behavior for fundamental types like `int64_t`. Users have had qualms with how I handle integers and other things due to all of Lua's numbers being doubles all the time: now, they don't have to beat up the library's defaults or make me write more configuration macros. They can just change it for their application, which is pretty great. It also keeps all you young whippersnappers in your own backyard, and _off my lawn, nyeh!_

Note that the code reduction is literally chopping out the namespaces and a bunch of other dirty template specializations. In fact, it's just writing your own functions now. And those functions don't have to sit in templates: you can put the declaration in a header, export them, and more! (The `Handler` is templated above because I'm lazy.) sol3 also features full-fledged commitment and support for a `<sol/forward.hpp>` header, which forward-declares everything necessary to make the above declarations work out (more compilation speed, nyeeoooom 🏃💨!).


# Okay, but Feature Complete doesn't mean Released...?

Ah, you got me. It's Feature Complete, but it's not Release-Ready. The code is fine, but:

- Documentation needs to be updated for how the new usertype stuff is done. This includes tutorials and API reference documentation. All examples are built as part of the tests though, which are passing! This means that the examples can serve as decent-enough documentation for the time being. Remember you can pop into the sol2 Discord: we're a pretty helpful bunch and help people not only get started but solve complex problems too! You can also @thephantomderp on twitter if you're a dork.
- The API reference documentation features me copy-pasting or hand-writing code that properly indicates the semantics of sol2. It would probably be better if I could just reference the actual block of constructors in the code, rather than creating (typo-filled) markup.
- I want to make a shiny sol3 logo. ... But I suck at drawing. So, that's a thing.
- The testing infrastructure sucks. Because my CMake skills suck.
- I need to tag vcpkg, Conan and maybe even build2 to make sure that they all have packages ready to go for sol3 at some point in the distant future (entirely optional, I'm really sick of package managers, meta-build systems, and build systems right now).

That's really it. There's some minor places where I need to Exorcise the Compiler Demon that is `std::tuple` and its many gross constructors, mostly by checking if `sizeof...(Args) == 2` in a `if constexpr` block and dispatching immediately to the handler without packing things into a `std::forward_as_tuple` first. I also can save a ton of compiler churn and bloat by making 1, 2, and 3-key partial specializations for my table proxy types so I no longer have to keep vomiting tuples upon tuples into these things for lazy-lookup key storage: I have not seen 4-deep proxy queries yet. (And if you have them without saving intermediaries, you deserve Compiler Demons that take root in your code and all the shame that comes with it!).

There is also the writhing leviathan that is the SFINAE-and-tag-dispatch-at-the-same-time things I have for function optimizations in sol2 that can probably be boiled down to some `if constexpr` so the `.set_function` call isn't so compile-time expensive, too.


# `if constexpr` ? But Derpstorm, what about my {old compiler here}?!

I gave a pretty long-standing heads up that sol3 will be going to C++17 tech, mostly to save me compile time speeds and sanity. Because sol3 is an open-source library, I get to do it for fun and not profit. Which translates to:

🎉 Pull requests welcome! 🎉

I have to finish school and I will be interning over the summer. sol2 has already seen excessive industry adoption, saved developers millions in useless Lua API churn and brought home plenty of fat cash stacks to shops around the globe. If your company needs a special snowflake version for C++11 or -- Lord in Heaven -- C++03, send me an e-mail and we can work out a contract for porting sol3 to your old, stable infrastructure. (Even if it's decrepit, I won't judge: I just spent the last week dealing with various flavors of Ubuntu and Debian and CentOS and their compilers and some Travis and Appveyor garbage. I hate the world as it is but I completely understand why you stick with your 4-years-out-of-support machine! ... A little. Okay, I'll be honest. There's some judgement. But only a bit, I swear!)

That's all for now. I'll be working on the documentation of sol3, some enhancements to a benchmarking framework so it's usable for all the crazy tooling stuff I want to do, and preparing to bother SG16's chair -- Tom Honermann -- and the legendary libogonek creator -- Robot -- with lots of Unicode Interface questions soon! And then after that it's school, and everything might go quiet.

[Or... will it? ♩~](/assets/img/2018-12-22/Dun dun dun.png)

Toodle-oo! ❤️
