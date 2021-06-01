# Feature Set

"What are we looking for?"


### Prior Art

There is a strong presence of prior tools:

- Assembly incbin
- xxd, CMake generators, python tools,
- (platform-specific) linkers

(All of them have various short comings.)


### Not the First Time

- `comp.std.c` feature requests from 30+ years ago
- Shows up in blog post / rant from a game developer every 3-5 years for the last 2 decades
- E-mails and interest from Lawrence Berkeley US National Labs, ISTEQ (Netherlands)
- Lamentation in Stack Overflow posts dating over a decade ago
- "how do I include a jpeg"


### The Basic Idea

```cpp
#include <assert.h>
#include <limits.h>

int main (int, char*[]) {
	const unsigned char jmp_sound[] = {
#embed <sdk/jump.wav>
	};

	// verify PCM WAV signature
	assert(jmp_sound[0] == 'R');
	assert(jmp_sound[1] == 'I');
	assert(jmp_sound[2] == 'F');
	assert(jmp_sound[3] == 'F');
	
	return 0;
}
```


### Simple

Effective way to get binary into programs:

- Uses flags, similar to `-I` (`-fembed-path`), to locate resources
- Behaves "as if" it generates an _initializer-list_ sequence
- Just an array: extensions/attributes for memory layout ordering still work
- Uni-directional: no hidden state/book keeping in preprocessor


### Simple, Flexible

```cpp
const unsigned char udata_str[] = {
	#embed "shader.txt"
	, 0
}; // null termination, simple

const unsigned char imaging_data[] __attribute__ ((aligned (8))) = {
	#embed "base_skeleton.tiff"
}; // attributes work fine too
```
