# Extrapolate Basis Operations

Transcoding, Validation, and Counting oh my!




# Transcoding

"Get text from Encoding A to Encoding B?"


### Simple Idea

We have `from_encoding` and `to_encoding` that are `encoding` objects

Both have `encode_one` and `decode_one` functions. If they:

- have a common `code_point` between them;
- and, can represent all the same values;
- and, does not error during the encode or decode step

we can always transcode!


### Setup

```cpp
struct ue_u8_encode_result {
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
ue_u8_encode_result transcode (utf_ebcdic from_encoding,  c_span input, 
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
			intermediate.begin(),
			from_result.output.begin()
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
	std::span<code_point_t> intermediate(buf);
	// ...
```


```cpp
	// ...
	for (;;) {
		auto result = encoding.decode_one(input,
			intermediate, handler, from_state);
		if (result.error_code != encoding_error::ok) {
			return false;
		}
		std::span<code_point_t> used(
			intermediate.begin(),
			from_result.output.begin(),
		);
		auto result = encoding.encode_one(used,
			output, handler, to_state);
		if (result.error_code != encoding_error::ok) {
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

	auto result = transcode(input, encoding, 
		output_buffer, encoding);

	return result.error_code == encoding_error::ok
		&& std::equals(input.cbegin(), input.cend(), 
			output.cbegin(), output.cend());
}
```


### Not recommended...

Memory consumption scales at `N` for the size of input.
But it is a fun exercise of the brain!




# Counting

"How many code units or code points will this operation yield?"


### 😜

This one is left as an exercise for the viewer.

<sub><sub><sub>(It's not hard or a trick, don't worry!)</sub></sub></sub>
