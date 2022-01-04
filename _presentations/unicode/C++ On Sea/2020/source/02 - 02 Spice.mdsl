# 🎺 Jazzing it Up 🎷


### iconv's API

```cpp
iconv_t iconv_open (const char* tocode, const char* fromcode);
size_t iconv (iconv_t cd, const char** inbuf, size_t* inbytesleft,
	char** outbuf, size_t* outbytesleft);
int iconv_close (iconv_t cd);
```


### What if we just... exposed the registry...?

```cpp
open_error registry_open (conversion_registry** p_registry,
	conv_heap heap);
open_error add_to_registry (conversion_registry* registry,
	const char* tocode, const char* fromcode,
	conv_conversion_function conversion,
	conv_open_function open_func, conv_close_function close_func);

open_error conversion_open (conversion** p_conversion,
	const char* tocode, const char* fromcode);
conv_error convert (conversion* conv,
	const unsigned char** p_input, size_t* p_input_size,
	unsigned char** p_output, size_t* p_output_size);
void conversion_close (conversion* conv);

void registry_close (conversion_registry* registry);
```


### Registry? Heap? open_function?? close_function???

Let's explain.


### Conversion Registry

We need a place to store encoding conversion:
- identified by unique pairwise to/from code
- to/from code is UTF-8 text; case-insensitive ASCII
- dashes, underscores, and blankspaces are ignored when comparing
- same to/from code overwrites previous encoding


### Conversion Registry

```cpp
int main (int argc, char* argv[]) {

	conversion_registry* reg;
	registry_open(&reg, {NULL}); // default heap

	// register utf-8 to utf-ebcdic conversion,
	// with default open/close functions
	add_to_registry(reg, "utf-8", "utf-ebcdic",
		&my_u8_ue_func, NULL, NULL);
	// overwrites last registration
	add_to_registry(reg, "UTF 8", "UTFEBCDIC",
		&other_u8_ue_func, NULL, NULL);

	registry_close(reg);

	return 0;
}
```


### Need a source of memory

Registry is dynamic, and therefore requires memory; embedded folk hate uncontrolled `malloc`; therefore:

```cpp
typedef struct __conv_heap {
	allocate_function allocate;
	reallocate_function reallocate;
	allocation_expand_function expand;
	allocation_shrink_function shrink;
	deallocate_function deallocate;
	void* user_data;
} conv_heap;
```


### open/close function?

```cpp
enum open_error {
	OPEN_ERR_OKAY = 0,
	OPEN_ERR_NO_CONVERSION_PATH = -1,
	OPEN_ERR_INVALID_CODE = -2,
	OPEN_ERR_ALLOCATION_FAILURE = -3,
};

typedef open_error (*conv_open_function)(conversion_registry* registry,
	conversion* conv,
	size_t* p_available_space,
	size_t* p_max_alignment,
	unsigned char** p_space);

typedef void (*conv_close_function)(void* data);
```


### Open has two jobs

1. if `conv == NULL`, simply report
	- the needed size from `p_available_space`
	- necessary alignment, taking into account any offset in `*p_space`
2. otherwise, actually fill in the required space at `*p_space` with state/data

(Close is just a plain ol' "undo what you did in `open`")


### Request Arbitrary Size; Fill in Arbitrary Data

Data is stored in the tail of the `conversion` opaque type

```cpp
// [           Implementation Bytes                |     N extra bytes   ]
// [ conversion_function | size_t | close_function | YOUR-OPEN-DATA-HERE ]
```


### That's it

Everything else is the same as iconv:
- get a conversion descriptor with the specified `to` + `from` codes
- call `convert` with it


### Anyone can register encodings

Implementations can insert "default" encoding pairs
- e.g. Wide and Narrow Locale Encoding conversions
- User can still override them if they "know better" later on
