# Syntax

> "It doesn't matter what color we paint the shed,  
so long as it's red."


### Expanding from Simple

- `comp.std.c` and WG14 Reflector discussion indicated need for more
  - "I need types that are not char/unsigned char."
  - "I need larger types."
  - "I want to map data to hardware table."
  - "I want optimal initialization of prefix trie."


### Expanded Idea

```cpp
#include <cassert>
#include <climits>

#define RIFF_LIMIT (2+2)

int main (int, char*[]) {
	const char sound_signature[] = {
#embed char RIFF_LIMIT <sdk/jump.wav>
	};

	// verify PCM WAV signature
	assert(sound_signature[0] == 'R');
	assert(sound_signature[1] == 'I');
	assert(sound_signature[2] == 'F');
	assert(sound_signature[3] == 'F');
	
	return 0;
}
```


### `char`? In MY Preprocessor?!

It gets more wild:

```cpp
#include <cstddef>

/* uhm?! */
size_t sizes[] = {
	#embed size_t "size_vals.bin"
};
```

How can the preprocessor know about types, typedefs, or type parsing? How can it generate the right values?


### Maybe use Bits/Bytes Instead?

```cpp
#include <cstddef>

#if /* Fragile Macro magic */
	#define SIZE_T_BYTES /* Stuff */
#else
	#define SIZE_T_BYTES /* Things */
	// ...
#endif

/* 🤔 */
size_t sizes[] = {
	#embed SIZE_T_BYTES "size_vals.bin"
};
```


### Problems with Types

Preprocessor does not know about types

- Think `Boost.Wave`:
  - can't compute size properly
  - marginally divorced from target environment
  - does not parse actual C++


## Problems with Bytes

Preprocessor also does not know about size of bytes

- Think `Boost.Wave`
  - "What's 8 ""bytes"" mean?"
  - DSP, exotic targets, etc.
  - Can only cleanly map the data to integral values


## Problems with Bits

Bits can work, but...

- Think `Boost.Wave`
  - "How do I initialize this exotic type with... 21 bits??"
  - Can only cleanly map the data to integral values
  - No other use cases allowed


### Our Savior: §5.2.4.2

> The values given below shall be replaced by constant expressions suitable for use in **`#if`** preprocessing directives. Their implementation-defined values shall be equal or greater to those shown.


### Limits, Preprocessor-Available

Including `<climits>` enables us to access sizes, minimums, and maximums for:

- `bool`
- `char`, `unsigned char`, `signed char`
- `short`, `unsigned short`, `signed short`
- `int`, `unsigned int`, `signed int`
- `long`, `unsigned long`, `signed long`
- `long long`, `unsigned long long`, `signed long long`


### Standard `#embed`, with Growth Room

Any `#embed` call that uses exactly the tokens of one of the integral types is supported

Otherwise, it's implementation-defined whether or not the `#embed` is supported  
(If it's not, diagnostic is required.)


### Solution

Always supported:

```cpp
const unsigned char icon[] = {
	#embed "gator.ico"
};
const int nums[] = {
	#embed int <numbers/vertices.bin>
};
const unsigned long long long_nums[] = {
	#embed unsigned long long <numbers/ext_vertices.bin>
};
```


## Solution

Implementation defined:

```cpp
#include <climits>

#if __cpp_pp_embed_classes
	const size_t sizes[] = {
		#embed size_t "numbers/sizes.bl"
	};
	const hardware_entry hw_tbl[]
	__attribute__ ((alias ("baddr11")) = {
		#embed hardware_entry "mxwll_40nm_x64.nvx"
	};
#else
	// time to write a `constexpr` parser for unsigned char data
	// Or just call Jason Turner / Ben Deane / Hana Dusíková :D
#endif
```
