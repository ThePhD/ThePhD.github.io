---
layout: post
title: A Weakness in the Niebloids
permalink: /a-weakness-in-the-niebloids
feature-img: "assets/img/2019-06-01/bandaged hand.jpg"
thumbnail: "assets/img/2019-06-01/bandaged hand.jpg"
tags: [C++, sol3, C++Now, extension points, ⌨️]
excerpt_separator: <!--more-->
---

Niebloids in range-v3 and other libraries has successfully demonstrated its prowess in avoiding ADL screwups while enabling those extension points of the same manner. But it's not for all extension point APIs: there are a few weaknesses in<!--more--> the range-v3 customization point model for both its interface and usability.

I had a C++Now 2019 talk on extension points called "The Plan for Tomorrow".

![The Plan for Tomorrow - a C++Now 2019 talk on Extension Points](/assets/img/2019-06-01/The Plan for Tomorrow.jpg)

While the video is not out yet, the slides are available [here](/presentations/sol2/C%2B%2B%20Now/2019/The%20Plan%20for%20Tomorrow%20-%20Compile-Time%20Extension%20Points%20in%20C%2B%2B.pdf). Before the video comes out, there's one topic I alluded to in my [C++Now 2019 Trip Report](/c++now-2019-trip-report) that I need to write down since at the time on stage I had forgotten to speak about it (not that I think I would have: I was just barely under time, and people already said I was talking fast)!


# But first: Niebloid?!

Niebloid is a catchy short hand named after Eric Niebler who crafted [range-v3](https://github.com/ericniebler/range-v3) and the Ranges TS, now officially `std::ranges` for C++20. It refers to a [set of requirements placed on algorithm invokables](http://eel.is/c++draft/algorithms.requirements#2). The effects written down in that clause are essentially only achievable by making a class that has an overloaded function call operator which is then instantiated as an object to use for the call. Alternatively, one can hold out hope for [Matt Calabrese's p1292](https://wg21.link/p1292) which -- I pray -- makes it for the C++23 cycle and saves us from having to go the long way around with function objects (a.k.a niebloids).

Still, niebloids were seen as the silver bullet of function-call based customization points that utilize ADL and is employed heavily in range-v3. But, there are some problems with it. Some of it I go through in my presentation at C++Now 2019, but there is one thing I did not get to talk about. It's a fairly minor interface thing, really, but something that I ended up valuing.


# Weakness: Explicit Templates

In my talk, I didn't cover an incredibly important feature for me when I wrote [the new sol3 customization points](https://github.com/ThePhD/sol2/blob/develop/examples/source/customization_multiple.cpp). Users need to be able to write `sol::stack::get<T>(...)`: notice the _template argument_ that needs to get specified and is not deduced at all. This means that it is impossible to use a function object call into extension point (a niebloid) for the customization points in sol3.

The only way to make it work is to have `get` be the result of instantiating a struct template: more precisely, a C++ Variable Template Niebloid:

```cpp
template <typename T>
struct niebloid {
	decltype(auto) operator () (/* args ... */ ) const noexcept {
		/* blah blah 
		customization point stuff here
		.. */
	}
};

template <typename T>
inline constexpr niebloid<T> customizable = niebloid<T>();

int main () {
	virtual_machine vm;
	int get_integer_from_state = customizable<T>(vm);
	/* use however */
}
```

Of course, this falls on its face when you want to sometimes specify template arguments, or otherwise not bother, as is the case with `sol::stack::push(lua_state, deduce_me);` and `sol::stack::push<MakeThisTypeInside>(lua_state, but, perfectly, forward, all, these, args, for, me);`. You would think this is a unicorn use case, but it actually [comes up in real usage](https://github.com/ThePhD/sol2/issues/814)! This is an additional parameter to consider when you want to use range-v3 style customization point design; does your customization point require template parameters? Is it optional? And so on, so forth.


### Slight Aside: Unicorn Proxies since sol2

As a side note, this is also why `sol::function`'s function call operator, `sol::protected_function`'s function call operator, and all of sol's table lookup functions have to return a proxy type that omni-converts to any type. If we could write `sol::function f = ...; f<T>(some, args, here);` and have the template arguments given to the `template <typename Ret, typename Args...> decltype(auto) operator()(Args&&...);` and not just error, we would be able to avoid this problem entirely and niebloids would be sufficiently powerful to cover all of the use cases. But, alas, it's C++: things can't just work! There's rules. Regulations! And of course, parsing shenanigans that obstruct this kind of syntax.

Granted, this isn't necessarily a problem: it's only a problem for _some_ use cases. Niebler's design space was for range-v3 and worked off of ideas behind things like `std::begin`, `std::swap`, etc. These types do not have template arguments passed explicitly to the call, so it's not something that could come up and not part of the surface area being managed by these customization points. Plus, like most things in C++, Niebler was essentially faced with creating a customization point design that could both defeat ADL and then leverage it: that's an exceedingly tall order, especially when several prominent voices in the Committee's Language Evolution group spend a lot of time shaking their heads at the Committee's Librarians when they talk about ADL being a problem.


# Any other weaknesses?

Some, but those are contained in my slides / talk! Go read the slides or watch the talk (when it comes out) if you have a chance to!

Ta-ta for now. 💚
