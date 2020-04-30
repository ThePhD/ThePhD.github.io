# Extrapolate Basis Operations

Transcoding, Validation, and Counting oh my!


# Transcoding

"Get text from Encoding A to Encoding B?"


### Simple Idea

Have: `from_encoding`/`to_encoding` that are `encoding` objects

Objects have `encode_one`/`decode_one` functions, so:

- have a common `code_point` between them?
- and, can represent all the same values?
- and, does not error during the encode or decode step?

we can always transcode!


![The basic conversion loop](resources/SimpleIdea.jpg)


### Setup

```cpp
struct ue_u8_transcode_result {
	c_span input;
	u8_span output;
	empty_struct& from_state;
	empty_struct& to_state;
	encoding_error error_code;
	bool handled_error;
};
```


### Transcode Implementation...

```cpp
ue_u8_transcode_result transcode (utf_ebcdic from_encoding,  c_span input, 
	utf8 to_encoding, u8_span output) {
	
	default_text_error_handler handler{};
	encoding_state_t<utf_ebcdic> from_state{};
	encoding_state_t<utf8> to_state{};

	using inbetween_t = encoding_code_point_t<utf_ebcdic>;

	inbetween_t buf[max_code_points_v<utf_ebcdic>];
	std::span<inbetween_t> intermediate(buf);

	for (;;) {
		// ...
```


#### Input: Decode to Code Points...

```cpp
		// ...
		auto from_result = from_encoding.decode_one(input,
			intermediate, handler, to_state);
		input = std::move(from_result.input);
		if (result.error_code != encoding_error::ok) {
			return { input, output,
				...,
				from_result.error_code,
				...
			};
		}

		std::span<inbetween_t> used(
			intermediate.data(),
			from_result.output.data()
		);
		//...
```


### What is that "`used`" calculation?

- First row: `intermediate`
- Second row: `from_result.output`

<img src="resources/Used.png" alt="Filled-in Boxes representing used space." />  


#### Output: Encode into new Code Units...

```cpp
		// ...
		auto to_result = to_encoding.encode_one(used,
			output, handler, to_state);
		output = std::move(to_result.output);
		if (to_result.error_code != encoding_error::ok) {
			return { std::move(input), std::move(output),
				...,
				to_result.error_code,
				...
			};
		}
		// ...
```


```cpp
		// ...
		if (input.empty()) {
			break;
		}
	}

	return { input, output, ... };
}
```




# Validation

"Is this what I want, or garbage?"


### Loop and Check

```cpp
bool validate (utf_ebcdic encoding, c_span input) {

	using code_point_t = encoding_code_point_t<utf_ebcdic>;
	using code_unit_t = encoding_code_unit_t<utf_ebcdic>;

	encoding_state_t<utf_ebcdic> from_state;
	encoding_state_t<utf_ebcdic> to_state;
	detail::pass_through_handler handler{};

	code_point_t buf[max_code_points_v<utf_ebcdic>];
	code_point_t out_buf[max_code_units_v<utf_ebcdic>];
	std::span<code_point_t> intermediate(buf);
	std::span<code_unit_t> output(out_buf);
	// ...
```


```cpp
	// ...
	for (;;) {
		auto from_result = encoding.decode_one(input,
			intermediate, handler, from_state);
		if (result.error_code != encoding_error::ok) {
			return false;
		}
		std::span<code_point_t> used(
			intermediate.data(),
			from_result.output.data(),
		);
		auto to_result = encoding.encode_one(used,
			output, handler, to_state);
		if (from_result.error_code != encoding_error::ok) {
			return false;
		}
		// ...
```


```cpp
		// ...
		c_span mirror_input(
			output.data(),
			to_result.output.data()
		);
		bool is_equal = std::equals(mirror_input.cbegin(),
			mirror_input.cend(), input.cbegin());
		if (!is_equal) {
			return false;
		}
		input = std::move(result.input);
		if (input.empty()) {
			break;
		}
	}
	return true;
}
```


### Look familiar?

We already defined Transcoding as:

- decode some code points
- if error, return with error
- else, take decoded code points and put into encode step
- if error, return with error
- else, loop back if input is not empty


### Laziest Possible Implementation

```cpp
bool validate (utf_ebcdic encoding, c_span input) {
	using code_unit_t = encoding_code_unit_t<utf_ebcdic>;

	detail::pass_through_handler handler{};
	std::vector<code_unit_t> output_buffer(input.size());
	std::span<code_unit_t> output(output_buffer);

	auto result = transcode(input, encoding,
		output, encoding);

	return result.error_code == encoding_error::ok
		&& std::equals(input.cbegin(), input.cend(),
			output.cbegin(), output.cend());
}
```


### Not recommended...

Memory consumption scales at `N` for the size of input.




# Counting

"How many code units or code points will this operation yield?"


### 😜

This one is left as an exercise for the viewer.

<sub><sub><sub>(It's not hard or a trick, don't worry!)</sub></sub></sub>


### (It's not Actually An Exercise)

API provides all of this!

- C++ Committee Paper version -  
  [https://wg21.link/p1629](https://wg21.link/p1629)
- Latest working draft -  
  [https://thephd.github.io/vendor/future_cxx/papers/d1629.html](https://thephd.github.io/vendor/future_cxx/papers/d1629.html)


```cpp
template <typename Input, typename Output, typename Encoding,
	typename State, typename ErrorHandler>
constexpr auto decode_into(Input&& input, Encoding&& encoding,
                           Output&& output, ErrorHandler&& error_handler,
                           State& state);

template <typename Input, typename Encoding,
	typename ErrorHandler, typename State>
constexpr auto decode(Input&& input, Encoding&& encoding,
                      ErrorHandler&& error_handler,
                      State& state);
```


```cpp
template <typename Input, typename Output, typename Encoding,
	typename State, typename ErrorHandler>
constexpr auto encode_into(Input&& input, Encoding&& encoding,
                           Output&& output, ErrorHandler&& error_handler,
                           State& state);

template <typename Input, typename Encoding,
	typename ErrorHandler, typename State>
constexpr auto encode(Input&& input, Encoding&& encoding,
                      ErrorHandler&& error_handler, State& state);
```


```cpp
template <typename Input, typename Encoding,
	typename EncodeState, typename DecodeState>
constexpr auto validate_as(Input&& input, Encoding&& encoding,
                        EncodeState& encode_state,
                        DecodeState& decode_state);

template <typename Input, typename Encoding,
	typename DecodeState, typename EncodeState>
constexpr auto validate(Input&& input, Encoding&& encoding,
                        DecodeState& decode_state,
                        EncodeState& encode_state);
```


```cpp
template <typename Input, typename Encoding, typename State>
constexpr auto encode_count(Input&& input, Encoding&& encoding,
                            State& state);

template <typename Input, typename Encoding, typename State>
constexpr auto decode_count(Input&& input, Encoding&& encoding,
                            State& state);
```


### Basic Use (Simple Overloads)

```cpp
#include <string>
#include <text_encoding>

int main (int, char*[]) {
	static_assert(std::text::validate_as(U"♥", std::text::literal_encoding{}),
		"Your current literal encoding that bakes characters into "
		"the final executable cannot handle hearts. "
		"Where is the love?! 😭");

	std::u8string utf8_emoji =
		std::text::encode(U“🐶”);
	// generally assume you want to go to UTF-8

	std::string ascii_emoji =
		std::text::encode(U“🐶”, std::text::ascii{},
			std::text::replacement_handler{});
	// actually just a '?', since ASCII can't handle it

	return 0;
}
```


### But...

Most importantly, about all of this...
