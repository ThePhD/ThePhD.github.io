# Speeding Up the Basis

Going faster than the Lucky 7


### Remember the Basis Algorithm?

for text `input` and writable `output` with objects `α` and `β`:

- if `input` is empty, return. otherwise:
  - call `α.decode_one` on `withinput` into `span` over  
  `code_point intermediate[max_code_points]`;
  - if error occurred, bail with error;
  - call `β.encode_one` with `intermediate` into `output`;
  - if error occurred, bail with error;
  - update `input` to move forward by # of `code unit`s read;


### Step 1: Cut Out Intermediates

for text `input` and writable `output` with objects `α` and `β`:

- if `input` is empty, return. otherwise:
  - if `text_transcode_one(...)` is available for parameters, call;
  - otherwise: {basic loop here!}


### `text_transcode_one`

Check if `text_transcode_one` free function exists:

If so? Use it...

```cpp
big5_u8_transcode_result text_transcode_one(
	c_span input, const big5_hkscs& from_encoding
	u8_span output, const utf8& from_encoding,
	ue_error_handler to_handler, u8_error_handler from_handler,
	bigs5_hkscs::state& from_state, utf8::state& to_state
);
```


```cpp
if constexpr (has_adl_transcode_one_v<Input, FromEncoding, ...>) {
	auto result = text_transcode_one(input, from_encoding,
		output, to_encoding,
		from_handler, to_handler,
		to_state, from_state);
	if (result.error_code != encoding_error::ok) {
		// something went wrong, bail
		return ...;
	}
	// use result, directly
}
else {
	// ... the basis loop
}
```


### Covers one-at-a-time `transcode`

- Improves speed of basic loop
- Helps `txt::transcode_view<utf8, bigs5_hkscs>` work faster



### Step 2: Enable Bulk

Bulk `validate`, `encode`, `decode`, etc. improvements?

- same principal, use an extension point!

For example...
