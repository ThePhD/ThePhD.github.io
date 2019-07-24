---
layout: post
title: Mutability and Conversions with Proxy References
permalink: /proxies-references-gsoc-2019
feature-img: "assets/img/pexels/birthday-gift-blue.jpg"
thumbnail: "assets/img/pexels/birthday-gift-blue.jpg"
tags: [C++, GSoC, iterators, proxies, design]
excerpt_separator: <!--more-->
---

Unlike the image, these proxy types are not the kind of gift an API designer wants under their Christmas Tree.<!--more-->

Several of us (Nathan Myers, Vincent Reverdy, Corentin Jabot and a few others) sitting at the Maredo Restaurant table during a lunch in Köln got around to discussing some of the challenges that `basic_dynamic_bitset` proposed. During the conversion, Vincent reminded me that I should absolutely write down everything that I learned. Having just returned from the C++ Standards Meeting at Köln, Germany, I figured it would be good to heed his advice. I won't talk about the actual Standards Meeting itself at the moment because the [reddit thread](https://www.reddit.com/r/cpp/comments/cfk9de/201907_cologne_iso_c_committee_trip_report_the/) and [Herb Sutter's post](https://herbsutter.com/2019/07/20/trip-report-summer-iso-c-standards-meeting-cologne/) have that covered! This post will serve as a precursor to a much more specific problem I have encountered on the lovely road to finishing up Phase 2 of my Summer of Code.



# Proxy wha-?

For the uninitiated, proxy references (and the special/wrapping iterators that make them) take advantage of the form and specification of the algorithms and containers in the standard to do some clever amount of work by (ab)using the syntax therein to perform some novel degree of work. As Titus Winters will tell you, "clever" is not a compliment when designing APIs, much less when designing in an API space that hands you footguns in a Buy-One-Get-Three-Free fashion like C++. And yet, proxy references and iterators can help make the expression of some fundamentally difficult problems all the more palatable and allow their usage in _some_ algorithms.

As such, this warrants an exploration of the design space... and the best way to start with exploring the design space, is by enumerating all of the problems therein!



### Wrapping Iterator / Proxy References

Wrapping iterators and proxy references -- e.g., those found on `std::vector<bool>` -- are the first layer of problems. They are fairly well documented in the standard library and elsewhere, due to the long heritage of iterator-based algorithms.

C++ generally upholds a fundamental rule for its container and iterator design: that the reference types gotten from dereferencing an iterator (e.g. `auto it = some_container.begin(); auto& val_ref = *it;`) should be real l-value references that point to some location in memory. This is why most people will write the following code for a vector when they first get their hands on templates:

```cpp
template <typename T>
void process_sundown_every_element(std::vector<T>& elements) {
	for (T& elem : elements) {
		elem = sundown(elem);
	}
}
```

This code, of course, holds until `bool` shows up for `T`. A `vector<bool>::reference` is produced now, which is not something that can be held in a plain ol' `bool& elem;`. Now you have the general gist of why everyone hates `vector<bool>`: `vector` promises to provide iterators that produce direct references, _except_ for `bool`. The code above just does not compile at all, given that lifetime extension is only applied to `const` references with `const auto& val_ref = *it;`. ... Unless you're on Microsoft's compiler with `/permissive-` turned off, where it'll happily lifetime-extend regular, non-const l-values as a compiler extension. Not sure if it does it to this code, but that is a compiler extension it has!


### The N:M problem

More complicated proxies exist. Some iterators do things that do not provide a 1:1 mapping from the wrapping iterator to the internal base. This means that, on top of making `auto& val_ref = *it;` fairly wrong (it needs to be `typename decltype(some_container)::reference` or similar), iterators and ranges which try to make a single value of `*it` become more than that run into severe problems.

This is solved for `std::vector<bool>` by making it so the returned `typename std::vector<bool>::reference` type is some inscrutable `_Bit_reference` type, which holds a reference to an integral type (a `std::size_t` on most platforms). It then performs the bit operations necessary to assign to one bit, and presents itself as a single `bool` bit value. Note that any iterator which references some sub-portion (or super-portion) of the actual `value_type` it is trying to use will run into this problem of having to return a not-`T&` for its `::reference` type, and will always roundhouse kick the unsuspecting user that wants `T&` to work in the face later.

Even `std::flat_map` might be engaged in this problem too because of the kind of `reference` type it has from its returned `iterator`s. `std::flat_map` is a double-container adaptor, and it returns a `std::pair<const key_type&, value_type&>`. The special `pair` type as a `reference` accommodates the split storage for keys and values. It tries to map 2 references into 1 returned proxy value (a 2:1 iterator, rather than a 1:64 iterator for something like `bit_iterator` over `std::size_t*` on a 64-bit system). This type does break some fundamental places in the design, where returning a copy of the `reference` type can be unsafe. The previous assumption of always getting out a directly reference-able type is also off the table, making `auto& val_ref = *it;` break pretty loudly as well (again, modulo compiler extensions).


### `swap` Problems

Another problem is fairly well documented: it is hard to write generic algorithms which respond properly to an inability to reference something directly. This makes swapping things (where `std::swap`'s general case is dependent on swapping two l-values) for a lot of the mutating algorithms a pain to write correctly, let alone agree with the assumptions underlying a basic assignment operation performed in an algorithm. The reason for this is because iterators who return proxy values as references typically do not obey the rules of assigning "directly" into the contents of their container.

Consider the case of a wrapping iterator `decoding_iterator<base_iterator>` which decodes some sequence of `typename base_iterator::value_type` and produces a composed `uint32_t` representing the decoded value. There is no way to form a proper, C++-blessed reference to that `uint32_t` because it was computed "on the fly" by invoking `*my_wrapping_iterator`. Even if we store the `uint32_t` on the iterator, this does not mean that assignment into a `uint32_t` will specifically encode back into the underlying iterator's sequence properly. This poses deep problems for a lot of types, ranging from `std::vector<bool>` to some of the ideas in [p1629 - Standard Text Encoding](/vendor/future_cxx/papers/d1629.html).

The addition of `std::ranges` has with it brought some fixes for that when writing algorithms: specifically, `iter_swap`. `iter_swap` existed before, but it was formally beefed up to be an explicit customization point by `std::ranges`. Doing this means that we can avoid the problem of `swap` needing l-value references (or something that behaves similarly to them) and instead we can swap what the 2 iterators point to "conceptually". It takes 2 iterators and swaps the contents, rather than two l-values: this is a level of indirection that proxy-making iterators can take advantage of to save from needing to address the fact that their wrappers may not support `swap` properly.


### When underlying `value_type`s are not equal...

`iter_swap` is only a panacea for wrapping iterators / proxy values when the proxy values always compute or consume in equal quantities. This breaks down when the values consumed are unequal. In the examples above, `std::vector<bool>::iterator`, `std::flat_map<K, V>::iterator`, and friends all work because every iterator does exactly the same task for `iter_swap`: even if they perform transformations, those transformations and equally and universally applied.

This does not hold true for the upcoming `std::text` and `std::text_view` mentioned in P1629 - Standard Text Encoding!

For example, consider a 7-bit decoding iterator, which walks a sequence of `std::byte` and employs the 7-bit decoding algorithm to produce an integer. Consider a (fictitious) decoding iterator:

```cpp
std::byte buffer[4]{0x82, 0x03, 0x01, 0x25};
/* ... */
integer7_iterator<std::byte*> bit7_it(my_buffer);
integer7_iterator<std::byte*> bit7_it_end();
```

The sequence of bytes pointed at, when decoded, yield the value `386` (uses 2 bytes to compute `386`). But, what could happen if we tried to make this iterator writable and do `*bit7_it = 1;`? Well, the last encoded value must be erased and then replaced with a value that takes only 1 byte (`1` uses only one 8-bit block of the 7-bit encoding).

And this is where the mayhem begins.

Suddenly, the iterator is the wrong level of abstraction: we need access to the original data, its end (iterator), the ability to shift all the integers after `1`'s insertion down to "erase" the second byte that is no longer exists, and all that implies. Writes can change the underlying container, which means the iterator is not enough.

This poses a real problem for algorithms like `std::sort`: **these** proxy values and references can't be `std::swap`'d around, even if `iter_swap` is used, because it requires a fundamental change to the storage rather than perfectly in-place operations. This means sorting your container of 7-bit integers must be done into a separate container after decoding is done, and certainly not with the basic `std::sort`.

This will pose interesting questions for the _contents_ of `std::text` and `std::text_view`. They will have to be mostly read-only iterators that do not support returning references -- even proxy references -- of any kind, and just raw values (e.g., `unicode_codepoint`/`unicode_scalar_value`/`extended_grapheme_cluster`). There will be no way to `std::sort` a `std::text`'s contents because even if we have all the proper Unicode collation rules and everything, the underlying storage does not map 1:1 with how we want to view the data (e.g., by Codepoints or Grapheme Clusters and such). Granted, I have never seen someone want to sort the entirety of a string by some arbitrary criteria: usually, they sort a collection _of_ strings, not the contents of each individual string itself. So, it will likely turn out to be a non-issue entirely!



# Alas...

There are more things to talk about, but that will come in a later post! I just wanted to get all of this sorted, because it's going to matter very heavily for the next few months for me. Keeping it written somewhere makes it easier to come back and see where I am wrong, and also to not forget things with the meat in my head.

Ta-ta for now. 💚
