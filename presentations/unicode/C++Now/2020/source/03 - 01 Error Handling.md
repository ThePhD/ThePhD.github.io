# Error Handling

The flexibility to do better


### Anatomy of a Handler

```cpp
ue_decode_result my_error_handler(
	const utf_ebcdic& encoding,
	ue_decode_result current_result,
	c_span unused_values
) {
	/* do whatever you want to the result */
	return current_result;
}
```


### "Do whatever you want"?

Yes; `result` object has all the information you could possibly need:

```cpp
struct ue_decode_result {
	c_span input;
	u32_span output;
	empty_struct& state;
	encoding_error error_code;
	bool handled_error;
};
```


### "Okay, but what **WOULD** you do?"

A lot:

- Replace every bad input with a replacement character
- Replace every bad _sequence_ with a replacement character
- Check for "incomplete" error, and try:
  - Store for networking buffering
  - Use normal handler to terminate output and end


### Oh.

And log stuff. 😁


### Replace bad sequence with replacement character

```cpp
bool utf_ebcdic_is_start( char code_unit );

ue_decode_result my_error_handler(
	const utf_ebcdic& encoding,
	ue_decode_result current_result,
	c_span unused_values
) {
	c_span& in_ref = current_result.input;
	auto first_valid = std::ranges::find_if(in_ref, &utf_ebcdic_is_start);
	in_ref = c_span(first_valid, current_result.input.end());
	// use standard replacement character after skipping things
	// to insert replacement character into output stream
	replacement_text_handler replacer{};
	return replacer(encoding, current_result, unused_values);
}
```


### Offload Incomplete Sequence into Buffer

```cpp
thread_local std::vector<char> thread_data_buffer;

ue_decode_result my_error_handler(
	const utf_ebcdic& encoding,
	ue_decode_result current_result,
	c_span unused_values
) {
	(void)encoding;
	encoding_error& ec_ref = current_result.error_code;
	if (ec_ref == encoding_error::incomplete_sequence) {
		thread_data_buffer.insert(thread_data_buffer.cbegin()
			unused_values.begin(), unused_values.end());
		ec_ref = encoding_error::ok;
	}
	return current_result;
}
```


### Standard Error Handlers

```cpp
class throw_handler;
class replacement_handler;
class assume_valid_handler;
using default_error_handler = replacement_error_handler;

template <typename Handler = default_error_hanler>
class ignore_incomplete_handler;
```
