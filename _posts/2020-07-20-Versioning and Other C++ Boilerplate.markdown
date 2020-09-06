---
layout: post
title: Versioning and other C++ Boilerplate
permalink: /versioning-and-other-boilerplate
feature-img: "assets/img/2020-07-20/meter-at-0000-1364700.jpg"
thumbnail: "assets/img/2020-07-20/meter-at-0000-1364700.jpg"
tags: [C++, C, ⌨️]
excerpt_separator: <!--more-->
---

I have not seen a "Best Practices" on how to handle configuration and similar for C and C++ libraries, so<!--more--> I'm going to write a guide on how to do this for C++ (and to a lesser extent, C) libraries! This will mostly cover how to handle C++ standard shenanigans, include file mishaps, and other silly things you have to check for versioning things!


# First things First

There's [an article by Arvid Gerstmann about how to deal with checking macros](https://arvid.io/2017/04/22/stop-using-ifdef-for-configuration/) properly. You do not have to read it because we are essentially going to demonstrate the exact same technique, as well as build upon it a little bit, as well as explain some bugs that show up from using define-based macro configuration techniques. In particular, the article advocates for a special "`USING`" macro:

```cpp
#define USING(x) ((1 x 1) == 2)
#define ON       +
#define OFF      -
```

You define internal configuration macros to either `ON` or `OFF` based on whatever pre-existing detection work you want to do. Here's a platform example:

```cpp
#if defined(_WIN32)
#	define PLATFORM_WINDOWS ON
#	define PLATFORM_MACOSX  OFF
#	define PLATFORM_LINUX   OFF
#elif defined(__APPLE__)
#	define PLATFORM_WINDOWS OFF
#	define PLATFORM_MACOSX  ON
#	define PLATFORM_LINUX   OFF
#else
	// all else belongs to the g l o r i u s p e n g u i n
#	define PLATFORM_WINDOWS OFF
#	define PLATFORM_MACOSX  OFF
#	define PLATFORM_LINUX   ON
#endif
```

This is important: if you have ever used a macro in `#if` or `#else` or similar blocks with the `define(...)` or `#ifdef` directives, it is impossible to tell if you forgot to define it or you defined it but to a bogus value. This is because the C Standard explicitly sanctions use of a macro in contexts like `#if` as "if it doesn't exist, the value is 0". For example, there's a bug in this code:

```cpp
namespace sol {

	class table {
	private:
		/* ... */
	public:
		table(lua_State* L, int index = -1) : table(detail::internal, L, index) {
#if defined(SOL_SAFE_REFRENCES) && SOL_SAFE_REFERENCES
			constructor_handler handler {};
			stack::check<basic_table_core>(L, index, handler);
#endif // Safety
		}

		/* ... */
	};

}
```

This can easily pass an eyeball test (and did pass my eyeball test while working on the [sol2 library](https://github.com/ThePhD/sol2)). It will also pass most tests. But this is not correct: look at the defines in monospace code next to each other:

```cpp
SOL_SAFE_REFRENCES
SOL_SAFE_REFERENCES
```

Yeah, I accidentally fat-fingered that `R` and `E`. Oops! But the preprocessor won't complain. By checking that it was defined first, well... you opted out of a proper existence check the compiler could make for you. If I change that to use the technique described above...

```cpp
namespace sol {
	class table {
	private:
		/* ... */
	public:
		table(lua_State* L, int index = -1) : table(detail::internal, L, index) {
#if USING(SOL_SAFE_REFRENCES)
			constructor_handler handler {};
			stack::check<basic_table_core>(L, index, handler);
#endif // Safety
		}

		/* ... */
	};
}
```

The compiler [will gently admonish you](https://godbolt.org/z/TGv78d):

```
error: missing binary operator before token "SOL_SAFE_REFRENCES"
```

Nice. It's obviously not the most clear error, but it stops your code from compiling when you do a silly, like I had done a silly.

Now, the _final_ approach I have been using is a little different; this is mostly because I want to be explicit about whether I want the thing on or off. Using my shenanigans library called `std0` as an example, here is a full example of how I set things up:

```cpp
#define STD0_IS_ON(_ON_OFF_SYMBOL)  ((1 _ON_OFF_SYMBOL 1) != 0)
#define STD0_IS_OFF(_ON_OFF_SYMBOL) ((1 _ON_OFF_SYMBOL 1) == 0)

#define STD0_ON  +
#define STD0_OFF -

#if defined(STD0_EXCEPTIONS)
	#if STD0_EXCEPTIONS != 0
		#define STD0_EXCEPTIONS_ STD0_ON
	#else
		#define STD0_EXCEPTIONS_ STD_OFF
	#endif // Someone has explicitly set support
#else
	#if !defined(__EXCEPTIONS) && !defined(_CPPUNWIND)
		#define STD0_EXCEPTIONS_ STD0_OFF
	#else
		#define STD0_EXCEPTIONS_ STD0_ON
	#endif // no known compiler macro is defined/undefined
#endif // Automatic Exceptions
```

And its use:

```cpp
class dynamic_array {
public:
	/* ... */

private:
	/* ... */

	constexpr void
	verify_element_capacity(size_type desired_element_capacity) const
	noexcept(alloc_access::handles_max_size::value) {
		if constexpr (!alloc_access::handles_max_size::value) {
			if (desired_element_capacity >= this->max_size()) {
#if STD0_IS_ON(STD0_EXCEPTIONS_)
				throw std::length_error("desired number of elements exceeds the"
					" maximum size"
					" this container can handle and the allocator does not have the"
					" handles_max_size type defined to std::true_type!");
#else
				assert(false && "desired number of elements exceeds"
					" the maximum size"
					" this container can handle and the allocator does not have the"
					" handles_max_size type defined to std::true_type!");
#endif // Exceptions on/off
			}
		}
	}
};
```

If I mess up using `STD0_EXCEPTIONS_` (because I spell it `STD0_EXEPTIONS_` or something equally tiny but preposterous), then I get a compiler warning:

```
fatal error C1012: unmatched parenthesis: missing ')'
	[std0\build\dynamic_array\profiling\shrink_to_fit
		\std0_dynamic_array_profiling_shrink_to_fit.vcxproj]
```

This is great! Messing up the usage of `STD0_EXCEPTIONS_` means unhappy customers, and we're not here for those noisy bug reports. It also helps when I accidentally spell `STD0_LIVE_FREE_OR_DIE_HARD_` as `STD0_LIVE_FREE_DIE_HARD_`, wherein a single conjunction turns the macro from an action movie plot to the U.S.'s general outlook on everything concerning poor people. This can result in the user getting the UB under the specified macro, or not, and is not something I should "accidentally" fudge in my library. It also prevents me from defining it to be a number or a string or something equally silly, or depending on the fact that I accidentally defined it to one of those things and then "fixing" it only to get weird bug reports of "well, like, it just worked, keep it that way!!".


### Not for the End User

Of course, the catch here is that this is only for _my_ internal code!

You'll notice that we use various knobs, bits, and bobs to make sure that the user-facing side is still easy to use. They only have to pass in a single macro definition to their compiler flags or define it, such as `STD0_EXCEPTIONS` (without the trailing underscore). This keeps it easy for them to do `-DSTD0_EXCEPTIONS=0`, while internally I get checked, consistent macro presence with the `suffixed_` version. The library user has an easy time, and I only have a chance to make the mistake once (in my version/configuration file), rather than vomiting that mistake out all over the codebase with consistently re-typed or copy-pasted `#if defined(FOO_BAR) && FOO_BAR != 0`.



# Beyond Configuration

But now, we need to do more than this! After all, it's not just configuration checking we need to do: there's other shenanigans afoot the `std::` ship we all take voyages on. For example, sometimes its helpful to check for the existence of headers (as autoconf and all its dastardly tools do at `make` time, bleugh!). This means using things like `__has_include` and, occasionally, `__has_builtin`. Chief among those scoundrel tendencies is one GCC employs quite frequently and, more recently, MSVC: the "we have this header, but we gated its contents internally on some arbitrary flags!"


### What?

Yep. Consider, for example, `<span>`. Nothing about its implementation requires C++20: not even close. You can get most of it with C++11, and tag the `std::byte` stuff along for the ride later if you really care. MSVC's implementation can be used outside of C++20, too, as it [properly swaps on whether concepts are available or not](https://github.com/microsoft/STL/blob/569efaba626f2b410e5bc8232ff063b87f1ff3dc/stl/inc/span#L370) (because their concepts implementation is broken in the most fantastically subtle ways and I have to routinely remind myself to restrain using it). And yet...

[No, no `<span>` for you](https://github.com/microsoft/STL/blob/569efaba626f2b410e5bc8232ff063b87f1ff3dc/stl/inc/span#L13).

This means that if you were to write the following code...

```cpp
#if defined(__has_include) && __has_include(<span>)
	#define STD0_WE_HAVE_SPAN_ STD0_ON
#else
	#define STD0_WE_HAVE_SPAN_ STD0_OFF
#endif
```

You would _still_ fail. The header exists, it will be included, but because you did not invoke the magical, fanciful `/std:c++latest` or `-std=[gnu|c]++[20|2x|2y]` (libstdc++ gates on `__cplusplus`, which is changed by the standard version flag) compilation will fail or the contents will just legitimately be empty after preprocessing. In fact, this very reason is why working with libc++'s `<optional>` and `<experimental/optional>` is [such a gigantic pain in the brain](https://en.cppreference.com/w/cpp/preprocessor/include):

```cpp
#if __has_include(<optional>) && (!defined (_MSC_VER) || __cplusplus >= 201703L)
	// don't include this unless we're sure it will be there for us
	// MSVC ships it early, might have gated it recently? Honestly who knows!
	#include <optional>
	#define have_optional 1
#elif __has_include(<experimental/optional>) && (!defined(__LIBCPP_VERSION) || (__cplusplus < 201703L))
	// okay: #error won't happen on libc++...
	#include <experimental/optional>
	#define have_optional 1
	#define experimental_optional 1
#else
	#define have_optional 0
#endif
```

libc++ has since removed the experimental header and does not ship such mistakes anymore, but it was definitely a slog to upgrade. (And then Apple bungled the release of `<variant>`, which was just (*chef's kiss*) a beautiful way to make [these checks](https://github.com/ThePhD/sol2/blob/7be51ebbef6719b6a99bdd7315acbb3ebe1b4c6c/include/sol/feature_test.hpp#L41) and [this macro](https://github.com/ThePhD/sol2/blob/7be51ebbef6719b6a99bdd7315acbb3ebe1b4c6c/include/sol/version.hpp#L59) necessary!)

So, how _do_ we check for feature sets in C++ and not just keep smashing a random collection of date and version checks together? C++20 offers a somewhat decent solution:



# `<version>`, our Lady and Savior

`<version>` is a new addition to C++ that is here to stop all of the excessive bull💩 developers have been dealing with for the past 40 years. It works a little like this:

```cpp
#include <version>

#if __cpp_lib_optional
	// it shipped!
	#include <optional>
#else
	// it did not ship: use TartanLlama's tl::optional!
	// ... honestly just use it anyways, the standard's one kinda sucks
#endif
```

And that's it. All that crap from above, reduced to a mere shadow of itself. If you are intent on supporting versions below this, you can do some basic checking to make sure the header at least exists:

```cpp
#if defined (__has_include) && __has_include(<version>)
	// we have it
	#include <version>
#endif

#if __cpp_lib_optional
	// it shipped!
	#include <optional>
#else
	// it did not ship: use TartanLlama's tl::optional!
	// ... honestly just use it anyways, the standard's one kinda sucks xD
#endif
```



# So much better!

And that's it; that's the way to do it unless you're interested in supporting older, more pitiful compilers and standard libraries. It's a pretty sweet addition and one I am infinitely thankful for: implementations know what they are and are not shipping and it should not be our jobs to book keep after any standard library's sloppiness. Now our macro checks are much smaller, and we can check both library features, and language ones! The language ones are much simpler to check for, because they are supposed to be predefined by the compiler:

```cpp
#if __cpp_concepts
	// I have concepts!
#else
	// ... 🙃
#endif
```

The C++ Committee also went far, far back in time to give every single macro a proper name. There's the ones [in the `<version>` header](https://en.cppreference.com/w/cpp/utility/feature_test), and ones that [belong to the language proper](https://en.cppreference.com/w/cpp/feature_test). You can grab a more complete list by [visiting Standard Document 6 (SD-6)](https://isocpp.org/std/standing-documents/sd-6-sg10-feature-test-recommendations) from the C++ Standards Committee (scroll close to the bottom to get to the part with the list of numbers you are interested in).

As you have noticed, my examples came from a mythical library called `std0`. But, well, it's not exactly a myth. I've been making a lot of claims lately about performance, finding a lower level library, and being backwards compatible. And so next, we're going to talk about how we leveraged the above -- and more -- to build a `std::vector` that's better than `std::vector` in a backwards-compatible, standards-compliant, drop-in replaceable way for C++. Well,

next time, anyhow.~

Toodle-oo. 💚
