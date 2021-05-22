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
#include <assert.h>
#include <limits.h>

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
#include <stddef.h>

/* uhm?! */
size_t sizes[] = {
	#embed size_t "size_vals.bin"
};
```

How can the preprocessor know about types, typedefs, or type parsing? How can it generate the right values?


### Numerics Instead?

```cpp
#include <stddef.h>

/* 🤔 */
size_t sizes[] = {
	#embed SIZE_T_BYTES "size_vals.bin"
};
```


### Problems with Both

Preprocessor does not know about types.  
(Can't compute size OR initialize properly if not an integer!)

Preprocessor also does not know about size of bytes.  
(Can't compute size!)


### Our Savior: §5.2.4.2

> The values given below shall be replaced by constant expressions suitable for use in **`#if`** preprocessing directives. Their implementation-defined values shall be equal or greater to those shown.


### Limits, Preprocessor-Available

Including this header enables us to access sizes, minimums, and maximums for:

- `char`, `unsigned char`, `signed char`
- `short`, `unsigned short`, `signed short`
- `int`, `unsigned int`, `signed int`
- `long`, `unsigned long`, `signed long`
- `long long`, `unsigned long long`, `signed long long`


### Standard `#embed`, with Growth Room

Any `#embed` call that uses exactly the tokens of one of the integral types is supported.

Otherwise, it's implementation-defined whether or not the `#embed` is supported. (If it's not, diagnostic is required.)


### Solution

Always supported:
```cpp
const int nums[] = {
	#embed int <numbers/vertices.bin>
};
const unsigned long long long_nums[] = {
	#embed unsigned long long <numbers/ext_vertices.bin>
};
```

Implementation defined:
```cpp
const size_t sizes[] = {
	#embed size_t "numbers/sizes.bl"
};
const struct hardware_entry hw_tbl[] = {
	#embed struct hardware_entry "mxwll_40nm_x64.nvx"
};
```


### Future Work?

Ability to specify exactly the bits, have numeric values from 0 to 2^56 read 56 bits at a time from file?

```cpp
const int64_t sizes[] = {
	#embed_bits 56 "numbers/sizes.bl"
};
```
