# Encoding Objects

And upon this rock, will they build their Encodings...


### A Strong Foundation

An encoding object is at minimum a collection of:

- 3 type definitions (`code_point`, `code_unit`, `state`)
- 2 static member variables (`max_code_points`, `max_code_units`)
- 2 functions (`encode_one`, `decode_one`)


### That's all you need.




### Example Encoding (Supporting Structures)

```cpp
#include <span>

struct empty_struct {};

using byte_span = std::span<std::byte>;
using c_span    = std::span<char>;
using u8_span   = std::span<char8_t>;
using u16_span  = std::span<char16_t>;
using u32_span  = std::span<char32_t>;

enum class encoding_error : int {
	ok                        = 0x00,
	invalid_sequence          = 0x01,
	incomplete_input          = 0x02,
	insufficient_output_space = 0x03
};
```


### Example Encoding (Result Types)

```cpp
struct ue_decode_result {
	c_span input;
	u32_span output;
	empty_struct& state;
	encoding_error error_code;
	bool handled_error;
};

struct ue_encode_result {
	u32_span input;
	c_span output;
	empty_struct& state;
	encoding_error error_code;
	bool handled_error;
};
```


### Example Encoding (Result Types)

```cpp
struct u8_decode_result {
	u8_span input;
	u32_span output;
	empty_struct& state;
	encoding_error error_code;
	bool handled_error;
};

struct u8_encode_result {
	u32_span input;
	u8_span output;
	empty_struct& state;
	encoding_error error_code;
	bool handled_error;
};
```


### Example Encoding (Error Handlers)

```cpp
using ue_decode_error_handler = std::function<
	ue_decode_result(const utf_ebcdic&, ue_decode_result, c_span)
>;
using ue_encode_error_handler = std::function<
	ue_decode_result(const utf_ebcdic&, ue_decode_result, u32_span)
>;

using u8_decode_error_handler = std::function<
	u8_decode_result(const utf8&, u8_decode_result, u8_span)
>;
using u8_encode_error_handler = std::function<
	u8_encode_result(const utf8&, u8_encode_result, u32_span)
>;
```


### Example Encoding - Exotic

```cpp
struct utf_ebcdic {
	using code_unit  = char;
	using code_point = char32_t;
	using state      = empty_struct;

	static constexpr inline std::size_t max_code_points = 1;
	static constexpr inline std::size_t max_code_units  = 6;
	
	ue_encode_result encode_one(c_span input, u32_span output,
		state& current, ue_encode_error_handler error_handler);
	ue_decode_result decode_one(u32_span input, c_span output,
		state& current, ue_decode_error_handler error_handler);
};
```


### Example Encoding - Common

```cpp
struct utf8 {
	using code_unit  = char8_t;
	using code_point = char32_t;
	using state      = empty_struct;

	static constexpr inline std::size_t max_code_points = 1;
	static constexpr inline std::size_t max_code_units  = 4;
	
	u8_encode_result encode_one(u8_span input, u32_span output,
		state& current, u8_encode_error_handler error_handler);
	u8_decode_result decode_one(u32_span input, u8_span output,
		state& current, u8_decode_error_handler error_handler);
};
```




### Seriously?

Seriously! Everything`*` can be built out of those 7:

- Bulk encode/decode/transcode ("go from A to B en masse...")
- Validation ("is this text correct?")
- Counting ("how many code points / code units are there?")
- Ranges ("I need code points for Freetype/harfbuzz")


`*` - almost ...


### Last One - Going Backwards

`decode_one_backwards`/`encode_one_backwards`

- `iterator`s obtained from `encoding_view`/`decoding_view`
  - `--it;` only works if these functions exist


### Specifically, `_one` unit of information

- saves higher levels of abstractions additional details
  - "insufficient output" errors never happen
  - no need to save state on encoding object between calls
  - gives end-user access to data to do as they want with (networking, etc.)


### Standard Encodings for C++23

```cpp
template <typename Encoding, std::endian Endian = std::endian::native,
	typename Byte = std::byte>
class encoding_scheme;

class ascii;
class narrow_execution;
class wide_execution;
class narrow_literal;
class wide_literal;
class utf8;
class utf16;
class utf32;
```


### "I need more"

Late C++23, Early C++26, or just plain old 3rd party libraries:

- WHATWG Suite of Encodings
  - "Minimum required to handle most web content"
- Legacy Encodings/Code Pages Library
  - Microsoft maintains quite a few as an extension
  - "Historian" types can collect these encodings easily

Also...


## You can make your own!

No special tricks, no secrets
