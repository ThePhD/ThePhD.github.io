---
layout: post
title: Fun with Storage - Allocators with small_bit_vector
permalink: /allocator-hell-small_bit_vector
feature-img: "assets/img/2019-10-14/dark-dice-gamble.jpg"
thumbnail: "assets/img/2019-10-14/dark-dice-gamble.jpg"
tags: [C++, allocators, bit, ⌨️]
excerpt_separator: <!--more-->
---

There is a dark place in the C++ Standard. A place even the greatest of implementers must mentally prep themselves before engaging. Welcome...<!--more-->




# To Allocator Hell

I see a fair amount of complaints about allocators and how complex they are. I have peeked at the specification and watched some talks on `pmr` allocators. None of that prepared me for actually implementing one of these things and using it at any level of reasonability. There are so many things you need to track, and while changes in C++11 made it so you can all but [gut your allocator](https://en.cppreference.com/w/cpp/memory/allocator) and implementers will take care of the details, well. Since I'm implementing `small_bit_vector`, _I am_ the implementer, and there's nobody left to defer the complexity to. In short:

_Awh geez_.




# Getting on the Allocator Train

It is at this point I must apologize to Zach Laine; I have seen his libraries where he strongly disavows the use of allocators and scoffs at the so-called benefits they purport in exchange for their obscene complexity. He has a few libraries where he just throws that stuff out the window, really: [Boost.Text](https://github.com/tzlaine/text) is one of them that I studied. I thought "well, if they are in the Standard, clearly they have a strong use, why would Zach throw them out?". And armed with this thought -- no, this... hubris -- I stepped onto the Memory Choo Choo headed straight for Allocator Hell.

I'm sorry, Zach. I should have listened to you, before I not only gazed but stepped off onto the `POCMA` platform.



## The Siren's Call: `allocator_traits`

In days gone by, allocators were truly a gnarly beast. From making sure all the `typedef`s were there to ensuring `rebind` properly swapped to the other allocator type, the complexity therein made it so implementing allocators was a chore the likes of which could not be taken by mere mortal creatures. Did you want a simple pool allocator? I hope you were ready to get snuggly with placement new, (pseudo-)destructors calls, object lifetime, trivial types, default initialization vs. default construction vs. aggregate initialization, and more. All on top of already trying to write a good way to put memory in the most cache-friendly places and keep fragmenting low.

Then, like an angel from the heavens, `allocator_traits` came down.

`allocator_traits` is a class type that simply gives static defaults to some 85%+ of the allocator machinery. Now, you only had to define `value_type`, `allocate`, and `deallocate`, and your job was done. All of the complexity was inside `allocator_traits`. I looked at my first C++11 allocator and laughed; this is it? This is all I had to define, this is all containers had to work with? Why, using this in my containers would be simple!




# Narrator: "But it wasn't simple."

It was easy to write one. That by no means meant it was easy to use one. `allocator_traits` may have made calling an allocator easy and not barfing template spew all over your error pane if you forgot something, but the sheer degree of involvement on the side of the implementation never changed. Things that sounded like far-progressed diseases took simple move and copy constructors and turned them into mental gymnastic sessions. Did you respect the `POCMA`? Did you catch the `POCCA`? Get up, come on get down with the sickness--



## And Down with the Sickness I Went

To start, there is no moving of allocators.

You may have noticed it when you looked at [all of `vector`'s constructors](https://en.cppreference.com/w/cpp/container/vector/vector). "Where's the `allocator&&` constructors?" A very valid and good question. It turns out, allocators can be stateful... but not the simple kind of stateful. Not a "respects-move-only" stateful. No, allocators were stateful in a strange way. I was so very confused, I puzzled over it again and again. How was I supposed to embed a buffer inside of a `std::unique_ptr` and then make a move-only allocator that strongly transferred ownership of both the `vector` and its owned buffer? It was not adding up, the compiler errors screamed, and I languished in Allocator Hell's fires until like rain from Heaven, the words of wisdom washed over me:

> "_[An Allocator is a Handle to a Heap](https://www.youtube.com/watch?v=0MdSJsCTRkY)..._"

Sage O'Dwyer, blessed amongst men, bearded like the Old Ones! Came his wisdom, cleansing me of my insecurities and puzzlement that had only become the fuel to Allocator Hell's fires. In a safe place did the wisdom of such a short and succinct phrase keep me, from the flames did it protect me, and my fingers once more could dance across the keyboard in glee.

Led by this simple advice -- allocators are meant to be handles -- I forged ahead. They are meant to be cheaply copied, or default constructed. That is the one true way, blessed by the Holy Standard. But then... I still had questions. What did this mean for move constructors? Move assignment? Assuredly, what would happen to my allocators then?


### `POCMA`, `POCCA` `SOCCC` ...?

Scary names. Or rather, scary acronyms: `propagate_on_container_move_assignment`, `propagate_on_container_copy_assignment`, `select_on_container_copy_construct`. Conspicuously missing from these `select`/`propagate` acronyms is `SOCMC`, which I imagined to be `select_on_container_move_construct`. To be perfectly honest, dear reader. I had no idea how I was supposed to handle these traits and functions. Plus, why was `SOCMC` missing? Here, I recalled a conversation in which Sage O'Dwyer himself participated. It was with one of the Blessed Few Implementers, Billy O'Neal, and the words I recalled even now came to my rescue before Allocator Hell consumed me in my ignorance.

I was not an active participant in the conversation. I was just listening while sitting down on the opposite side of the table at CppCon. Quietly, Blessed Implementer Billy spoke of doing the very thing I was struggling with:

> ... You just move the allocator when you move construct. And that's the only case where you do that; every other case you have to copy when they're not equal, or leave it default constructed. POCCA is false or they are not equal, you need to manually assign then clear...

I don't remember the rest. But these words were enough. Thusly, I began to write:

```cpp
small_bit_vector& operator=(const small_bit_vector& right) {
	allocator& left_alloc = this->get_allocator();
	allocator& right_alloc = right.get_allocator();
	if constexpr (this_alloc_traits::propagate_on_container_copy_assignment::value) {
		// must 'propagate' (assign) if different
		if (left_alloc != right_alloc) {
			// ... Wait, what do we do here?
		}
		left_alloc = right_alloc;
	}
	else {
		// do not copy:
		// just let `this` alloc
		// be itself!
	}
	this->assign(__right.cbegin(), __right.cend());
	return *this;
}
```

Now, what did I do if the allocators do not compare equal? Well, not being equal means that they are -- apparently -- handles to _different_ heaps. What this means is that if I were to assign from the right to the left, and then overwrite things from the left container, I've essentially orphaned the `left`'s (`this`'s) memory, because they point to different heaps and have different valid operations!

... So I need to handle the one thing that normally would not happen in this case, which is I need to destroy and get rid of all the memory from `left` before writing over it with the content of `right`'s allocator:

```cpp
small_bit_vector& operator=(const small_bit_vector& right) {
	allocator& left_alloc = this->get_allocator();
	allocator& right_alloc = right.get_allocator();
	if constexpr (this_alloc_traits::propagate_on_container_copy_assignment::value) {
		// must 'propagate' (assign) if different
		if (left_alloc != right_alloc) {
			// destroy elements, deallocate memory
			// before allocator gets written over!!
			this->internal_clear_and_deallocate();
		}
		// now safe to plow 
		// over container contents
		left_alloc = right_alloc;
	}
	else {
		// do not copy:
		// just let `this` alloc
		// be itself!
	}
	this->assign(__right.cbegin(), __right.cend());
	return *this;
}
```

I will be perfectly honest: I have no idea if this is actually how the copy assignment operator is supposed to be handled. It seems like that was the intent from the standard, and this falls in line with what my patchy memory of the CppCon Hallway Track Talk that Blessed Implementer O'Neal (perhaps unknowingly) handed out. And it... _seems_ to work! Steadily, I was making my way through Allocator Hell.

There is apparently one problem here, though.

See, there's more magic in `allocator_traits`: one of the member types is called `is_always_equal`. And, what's more, is that if this type exists and it is `std::true_type`, then your allocator can leave off defining any comparison operators entirely. That means the above code is completely ill-formed for one of those allocators, since it will attempt a comparison on propagation! So, another slight edit:

```cpp
small_bit_vector& operator=(const small_bit_vector& right) {
	allocator& left_alloc = this->get_allocator();
	allocator& right_alloc = right.get_allocator();
	if constexpr (this_alloc_traits::propagate_on_container_copy_assignment::value) {
		// must 'propagate' (assign) if different
		if constexpr (!this_alloc_traits::is_always_equal::value) {
			// allocators are not alway equal...
			if (left_alloc != right_alloc) {
				// destroy elements, deallocate memory
				// before allocator gets written over!!
				this->internal_clear_and_deallocate();
			}
		}
		// now safe to plow 
		// over container contents
		left_alloc = right_alloc;
	}
	else {
		// do not copy:
		// just let `this` alloc
		// be itself!
	}
	this->assign(__right.cbegin(), __right.cend());
	return *this;
}
```

There. _Now_ it's looking like a proper container assignment! The wisdom of the ancients for allocators has prevailed. Still, we're not done: there's a move assignment to implement too. Move is more interesting because, if possible, we want to steal the container's data. When you move a typical `std::vector<int>` from one side to another, it does not just copy. It _steals_ the guts in most cases. So even if the conditionals used remain largely the same, the work done changes because we want to steal details, or else move the individual contents out if possible. Also note that we always have to clear the internal state first before doing a steal, because we need to use our current allocator to do that before writing over it:

```cpp
small_bit_vector& operator=(small_bit_vector&& right) {
	allocator& left_alloc = this->get_allocator();
	allocator& right_alloc = right.get_allocator();
	if constexpr (this_alloc_traits::propagate_on_container_copy_assignment::value) {
		this->internal_clear_and_deallocate();
		// take allocator AFTER deallocating
		// this memory
		left_alloc = std::move(right_alloc);
		this->internal_steal(std::move(right));
	}
	else {
		// No propagation
		if constexpr (this_alloc_traits::is_always_equal::value) {
			this->internal_clear_and_deallocate();
			this->internal_steal(std::move(right));
			// do not steal allocator, no propagation!
		}
		else {
			// Awh geez: check at runtime
			if (left_alloc == right_alloc) {
				// okay! Runtime equal, steal stuff
				this->internal_clear_and_deallocate();
				this->internal_steal(std::move(right));
				// do not steal allocator, no propagation!
			}
			else {
				// ... welp, slow path time...
				// move elements
				this->assign(std::make_move_iterator(right.begin()),
					std::make_move_iterator(right.end()));
			}
		}
	}
	return *this;
}
```

Remember that allocators must be `copyable` and not movable, so you do not want to do `left_alloc = std::move(right_alloc);`. Also keep in mind that even after you move out of a container (steal the guts), you can still "revive" that container by doing things like calling `.clear();` and then `.push_back(value);`! Always, keep in mind Sage O'Dwyer's words: _Allocators are a Handle to a Heap_.

One thing that still lingers at the back of my mind is whether or not you are supposed to `right.clear()` in the case where the slow path is taken and you assign from the `make_move_iterator`s. But, alas... I will need to do peer more closely at the standard and its wording or re-watch the videos available a few more times to understand if that is the case.




# So Are We Out of Allocator Hell?!

It would seem like it. Out of the dangerous Valley of Memory Management, away from the fires and into the glorious rolling hills and meadows away from these low-level, gory details. But, when starting some preliminary performance testing, I noticed something... peculiar. There was a slight performance degradation beyond what I had anticipated, even in my somewhat shoddy initial implementation. My flame graphs had me once more turn around and confront the hell that neither the Blessed Implementers nor Arthur's talk exactly prepared me for...



## Default Initialization

Or more appropriately, the lack thereof. In working with bits, you work with -- generally -- built-in integral types. As I [briefly alluded to in my last post](/small-bit-vector), such built in types have a difference between its regular initialization, and this weird, security-vulnerability-inviting default initialization state:

```cpp
int main () {
	// value: 0
	int well_defined = int();
	// value: ???
	int undefined;

	return 0;
}
```

And what goes on inside the allocator would be something like this, given some "blob" of memory:

```cpp
int main () {
	unsigned char well_defined_blob[sizeof(int)];
	new int(&[well_defined_blob]) int ();
	// value: 0
	int& well_defined = *(int*)&well_defined_blob[0];

	unsigned char indeterminate_blob[sizeof(int)];
	new int(&[undefined_blob]) int;
	// value: ??????????
	int& undefined = *(int*)&undefined_blob[0];

	return 0;
}
```

Normally, this is a serious problem. Uninitialized variables cause a lot of hard to debug errors because the memory in them can be whatever the hell the operating system or process left there last. They also become the source of a lot of security issues, with exploiters finding the quickest path to getting your code to touch uninitialized memory and rocket off into unexpected places so they can execute an attack on your code. Unfortunately for my speed OCD,

I could see the difference.

Was it much? Not really. Unfortunately, I'm not exactly the only customer of this type: I don't know who would use this, and where. That means that I very well may be imposing a cost upon my end user that they did not ask for, nor want to pay for. What makes this worse is that -- as the provider of a generic container taking any old allocator -- there was no way to choose the blob of undefined garbage as the initialization pattern. Plus, by the C++ Abstract Virtual Parameterized Machine™, indeterminate values invoke undefined behavior. To give an example:

```cpp
#include <cassert>

int main () {

	unsigned i;
	i |= 0x01u;
	bool is_true = (i & 0x01u) == 0x01u;
	assert(is_true); // is it??

	return 0;
}
```

One of the few Living C++ Compilers weighed in out of the blue when I expressed my dilemma:

> We have no such thing as a partially-determinate value -- there are only "real" integer values and (fully) indeterminate values in the abstract machine's model, we don't have bit-level tracking. So that example can't work in the C++ abstract machine. Real implementations end up supporting cases like that example because they show up in the middle-end for bit-fields.
> 
> [[basic.indet]p2](http://eel.is/c++draft/basic.indet#2) was intentionally stricter than what current implementations need; we didn't want the full complexity of bit tracking and value agnosticism and whatever else as part of the abstract machine rules.

That's a pretty fair assessment of the situation. But that still doesn't help _me_! The bits I let users access through `small_bit_vector` are well-defined because they had to be written to using one of the functions in order to be accessible. The problem at the core becomes that the following call does not let me differentiate between "I want indeterminate, default initialization" and "I want the default constructor to be called":

```cpp
void push_back () {
	internal_pointer_type memory_ptr = ...;
	bool at_bit_boundary = ...;
	if (at_bit_boundary) {
		this_alloc_traits::construct(memory_ptr);
	}
}
```

No arguments just ends up calling `new (memory_ptr) value_type();`. There's no way -- generically -- to grease the allocator's palms and tell it to do `new (memory_ptr) value_type;` instead. C++ didn't let me give you the performance! It was the allocator's fault! The same way it was the allocator model's fault that there was no `reallocate` function! C++'s fault, not my own; I wasn't responsible for Allocator Hell, I didn't make the rules! Just a cog in the machine. A rung on the ladder, a step on the stairs. And yet...

And _yet_.




# Tossing and Turning

There... there must be another way. Perhaps there was. There must be some trick, some way to do default, garbage initialization of the bits the user would work with. To get that last handful of %. It came to me, in the dead of the night. I sat right up in bed, with that little idea. There was no chatroom of C++ experts around this time. Even the Europeans were still sleeping at this ungodly hour. I shuffled to the desk and took a seat. The brightness of the monitor hurt my eyes for a brief moment.

In my imagination, it grew. Firey, burning condemnation that was much stronger than Allocator Hell could ever bring to task. After all, the casual conversation of Blessed Implementers and words from Arthur O'Dwyer could save me from that. This... this was a much more serious infraction. One that cut against the heart of the very abstract machine itself. The condemnation for doing this...

I clicked on the `itsy.bitsy` shortcut. The comforting deep blue of VSCode came up, a sharp contrast to the condemnation I felt rising in my soul, my very spirit averse to what I was going to do. A consistent effort to always stay inside the rules, to always do things properly and correctly... and yet, here I now was. Contemplating engaging in something so heinous the action itself could beget demons into the world.

The editor screen populated. A little scrolling and I saw it:

```cpp
__alloc_traits::construct(__mem_alloc, __storage_pointer);
```

A simple line. Decorated in the usual standard library double underscores. The one line that caused that noticeable little performance blip. The obstruction to a greater, higher purpose of maximum and absolute speed. This...

**thorn** in my side.

I began to type. I needed a conditional.

```cpp
if constexpr () {

}
else {
	__alloc_traits::construct(__mem_alloc, __storage_pointer);
}
```

What was the greater humiliation, dear reader? That performance loss, or a minor infraction against a machine already admitted to be "stricter than current implementations need"? Are we not living and breathing creatures? What is a pile of legalese a thousand pages long in print and a collection of zeroes and ones compared to our flesh and bones? It has no feelings. I heard Alexandrescu talk about it: Speed is found in the minds of the people! Mortals, human beings, their feelings matter, the performance is for them.

I did this for them! Surely you understand, dear reader?

```cpp
if constexpr(::std::is_trivial_v<__base_value_type> && ) {

}
else {
	__alloc_traits::construct(__mem_alloc, __storage_pointer);
}
```

If you knew how this kept me up, you would not judge me. You've done things just like this, haven't you? You'll get what I'm doing, why I'm doing this. You saw what I went through! Just to even get this far with allocators... only for the model to penalize me in a way writing naked C++ wouldn't. That just wouldn't do.

Right, dear reader?

```cpp
if constexpr(::std::is_trivial_v<__base_value_type> 
	&& not __is_detected_v<>
) {

}
else {
	__alloc_traits::construct(__mem_alloc, __storage_pointer);
}
```

It was the tiniest loophole. One opened by my C++11 predecessors... it wasn't even my fault. You didn't _have_ to have a `construct` function, right? You would not be able to tell. C++20 removed all of those pesky functions off of the default allocator, too: assuredly, they knew this to be true. If a `construct` function doesn't exist...

Just a tree in a forest nobody is around for, yes?

```cpp
if constexpr(::std::is_trivial_v<__base_value_type> 
	&& not __is_detected_v<__allocator_traits_construct_call, allocator, __base_pointer>
) {

}
else {
	__alloc_traits::construct(__mem_alloc, __storage_pointer);
}
```

I don't want to just be fast. If I can write this code faster by hand, what's the point of writing the abstraction at all? This is serious code. Meant for production. Meant for High Performance Computing. Destined for greatness! Fast is not good enough. Faster is a consolation prize for the weak.

Only the **fastest** code.

```cpp
if constexpr(::std::is_trivial_v<__base_value_type> 
	&& not __is_detected_v<__allocator_traits_construct_call, allocator, __base_pointer>
) {
	new (__storage_pointer) __base_value_type;
}
else {
	__alloc_traits::construct(__mem_alloc, __storage_pointer);
}
```

I finished the deed. My fingers left the keyboard for a moment as I stared. The fires in my imagination receded, and I was left with just the quiet of the night and the loud hum of my old computer's fan. I was not smote in my seat. My hard drive was not reformatted. It's just a tiny little bit of it, dear reader. And, well... come now. A little indeterminate value, a tiny tiny bit of "Undefined" Behavior...

![Save Button](/assets/img/2019-10-14/save.png)

It'll never hurt anyone...

💚
