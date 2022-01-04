# Fast?


### 🐌

Yes it is:

- maximally safe
- optimally compatible

No it is not fast


### There are Faster Methods

Take advantage of...

- Shortcut round tripping without common code point
- Take advantage of additional information on encoding object
- [`T*, T*`) or [`T*, size`) ranges to do SIMD / fast computation


### Example: Transcoding

Remember this?

```cpp
		auto result = from_encoding.decode_one(input,
			intermediate, handler, from_state);
		if (result.error_code != encoding_error::ok) {
			return false;
		}
		std::span<code_point_t> used(
			intermediate.begin(),
			from_result.output.begin(),
		);
		auto result = to_encoding.encode_one(used,
			output, handler, to_state);
		if (result.error_code != encoding_error::ok) {
			return false;
		}
```


### What if UTF-EBCDIC ⟺ UTF-8?

No need to go through `code point`

- UTF-EBCDIC special intermediate step
  - Called "UTF-8 Mod" before Look-up Table
  - Can take advantage of this to translate quickly


### `text_transcode_one`

Check if `text_transcode_one` free function exists:

If so? Use it...

```cpp
ue_u8_transcode_result text_transcode_one(
	c_span input, utf_ebcdic from_encoding
	u8_span output, utf8 from_encoding,
	ue_error_handler to_handler, u8_error_handler from_handler,
	utf_ebcdic::state& from_state, utf8::state& to_state
);
```



```cpp
		auto result = text_transcode_one(input, from_encoding,
			output, to_encoding,
			from_handler, to_handler, 
			to_state, from_state);
		if (result.error_code != encoding_error::ok) {
			return false;
		}
```


### "Encoding Object" contains Extra Information

Consider a mystical, magical `any_encoding`

- Type-erases the encoding and stores it inside of itself
  - On `enc.decode_one`, unwraps `this` and calls real `stored_encoding.decode_one`
  - On `enc.encode_one`, unwraps `this` and calls real `stored_encoding.encode_one`
- And so on...


### Remember those loops?

Think about `decode_one` for `any_encoding` for 1,000,000 code points.


### Remember those loops?

- Loop (`* 1,000,000`)
  - Get to call (OVERHEAD)
  - Unwrap, do real call
  - Exit call (OVERHEAD)

`(2 * (OVERHEAD)) * 1,000,000`


# 😬


### Can do better!

Check if `enc.decode` function exists on object:

- If it does, call that instead of running `enc.decode_one` in a loop
  - Unwrap
  - Do 1,000,000 code points without overhead
  - Exit function



# Much Nicer Flow

### Much Better Performance 👍


### Applies in other places

- Bulk `text_(de|en|trans)code(...)` ADL extension points
  - Developer can implement if they know better than us  
    (UTF-EBCDIC ⟺ UTF-8)


### Detect `span` or `contiguous_range`?

Use C functions 😎

<img src="resources/C - proposed functions.png" alt="Proposed list of C transcoding functions." width="50%" height="50%"/>


### Many other instances of this

- `enc.validate_one(...)` method instead of wasteful `enc.decode_one(...)`
  - Also bulk `text_validate(...)` ADL extension point
  - E.g.: [Daniel Lemire's Work](https://lemire.me/blog/2018/10/19/validating-utf-8-bytes-using-only-0-45-cycles-per-byte-avx-edition/)
- `enc.encode_count_one(...)` method instead of wasteful `enc.encode_one(...)`
  - Also bulk `text_encode_count(...)` ADL extension point
