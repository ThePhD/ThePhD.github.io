### Previous Work

Part 1 (Catch it [here](https://www.youtube.com/watch?v=BdUipluIf1E))  
Part 2 (Catch it [here](https://www.youtube.com/watch?v=FQHofyOgQtM))  
Part 3 (Catch it [here](https://www.youtube.com/watch?v=FQHofyOgQtM))  

<a href="https://www.youtube.com/watch?v=BdUipluIf1E"><img src="resources/Previously0.jpg" alt="Linked Picture of Previous YouTube Video on Unicode for C++23, Part 1" width="30%" height="30%" /></a> <a href="https://www.youtube.com/watch?v=FQHofyOgQtM"><img src="resources/Previously1.png" alt="Linked Picture of Previous YouTube Video on Unicode for C++23, Part 2" width="30%" height="30%" /></a> <a href="https://www.youtube.com/watch?v=w4qYf2pvPg4"><img src="resources/Previously2.png" alt="Linked Picture of Previous YouTube Video on Unicode for C++23, Part 3" width="30%" height="30%" /></a>


### Last Time, We Did Math™

There exists a small set of basis operations  
for an encoding `α` or `β` that allows us to:

- encode (code points of `α` ➡ code units of `α`) text
- decode (code units of `α` ➡ code points `α`) text
- transcode (code units of `α` ➡ code units of `β`) text
- validate code units in text ("is this UTF-8?")
- validate code points in text ("can these code points be ASCII?")
- count code units from code points ("# Big5 bytes in these code points?")
- count code points from code units ("# code points in this UTF-8?")


### Lucky 7 - An Example Encoding Object

```cpp
struct utf8 {
	using code_unit  = char8_t; // #1
	using code_point = char32_t; // #2
	using state      = empty_struct; // #3

	static constexpr inline std::size_t max_code_points = 1; // #4
	static constexpr inline std::size_t max_code_units  = 4; // #5
	
	u8_encode_result encode_one(u32_span input, u8_span output,
		state& current, u8_encode_error_handler error_handler); // #6
	u8_decode_result decode_one(u8_span input, u32_span output,
		state& current, u8_decode_error_handler error_handler); // #7
};
```


### Lucky 7 - An Example Exotic Encoding Object

```cpp
struct big5_hkscs {
	using code_unit  = char; // #1
	using code_point = char32_t; // #2
	using state      = empty_struct; // #3

	static constexpr inline std::size_t max_code_points = 2; // #4
	static constexpr inline std::size_t max_code_units  = 2; // #5
	
	big5_encode_result encode_one(u32_span input, c_span output,
		state& current, big5_encode_error_handler error_handler); // #6
	big5_decode_result decode_one(c_span input, u32_span output,
		state& current, big5_decode_error_handler error_handler); // #7
};
```


### Supporting Types - Error Codes

```cpp
using c_span = std::span<char>;
using u8_span = std::span<char8_t>;
using u32_span = std::span<char32_t>;

struct empty_struct {};

enum class encoding_error : int {
	ok                        = 0x00,
	invalid_sequence          = 0x01,
	incomplete_input          = 0x02,
	insufficient_output_space = 0x03
};
```


### Supporting Types - Result Types

```cpp
struct big5_decode_result { // u8_decode_result for utf-8
	c_span input; // u8_span for utf-8
	u32_span output;
	empty_struct& state;
	encoding_error error_code;
	bool handled_error;
};

struct big5_encode_result { // u8_encode_result for utf-8
	u32_span input;
	c_span output; // u8_span for utf-8
	empty_struct& state;
	encoding_error error_code;
	bool handled_error;
};
```


### Supporting Types - Error Handlers

```cpp
using u8_decode_error_handler = std::function_ref<
	u8_decode_result(const utf8&, u8_decode_result, u8_span)
>;
using u8_encode_error_handler = std::function_ref<
	u8_encode_result(const utf8&, u8_encode_result, u32_span)
>;
using big5_decode_error_handler = std::function<
	big5_decode_result(const big5_hkscs&, big5_decode_result, c_span)
>;
using big5_encode_error_handler = std::function<
	big5_encode_result(const big5_hkscs&, big5_encode_result, u32_span)
>;
```


### Supporting Types - Error Handlers (better, C++23?)

```cpp
using u8_decode_error_handler = std::function_ref<
	u8_decode_result(const utf8&, u8_decode_result, u8_span)
>;
using u8_encode_error_handler = std::function_ref<
	u8_encode_result(const utf8&, u8_encode_result, u32_span)
>;
using big5_decode_error_handler = std::function_ref<
	big5_decode_result(const big5_hkscs&, big5_decode_result, c_span)
>;
using big5_encode_error_handler = std::function_ref<
	big5_encode_result(const big5_hkscs&, big5_encode_result, u32_span)
>;
```


### The Basis

```cpp
struct big5_hkscs {
	using code_unit  = char; // #1
	using code_point = char32_t; // #2
	using state      = empty_struct; // #3

	static constexpr inline std::size_t max_code_points = 2; // #4
	static constexpr inline std::size_t max_code_units  = 2; // #5
	
	big5_encode_result encode_one(u32_span input, c_span output,
		state& current, big5_encode_error_handler error_handler); // #6
	big5_decode_result decode_one(c_span input, u32_span output,
		state& current, big5_decode_error_handler error_handler); // #7
};
```


![The basic conversion loop](resources/SimpleIdea.jpg)


### Lucky 7 Basis Algorithm

for text `input` and writable `output` with objects `α` and `β`:

- if `input` is empty, return. otherwise:
  - call `α.decode_one` on `withinput` into `span` over  
  `code_point intermediate[max_code_points]`;
  - if error occurred, bail with error;
  - call `β.encode_one` with `intermediate` into `output`;
  - if error occurred, bail with error;
  - update `input` to move forward by # of `code unit`s read;


### The Lucky 7 Basis Can Build Everything 

[Part 3](https://www.youtube.com/watch?v=w4qYf2pvPg4) successfully described a Lucky 7 definition to build...

- `encode(...)`
- `decode(...)`
- `transcode(...)`
- `validate(...)`/`validate_as(...)`
- `encode_count(...)`/`decode_count(...)`
