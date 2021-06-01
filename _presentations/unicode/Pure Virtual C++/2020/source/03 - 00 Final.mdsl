# The _Basis Never Changes_


### Lucky, Lucky 7

```cpp
struct utf8 {
	using code_unit  = char8_t;
	using code_point = char32_t;
	using state      = empty_struct;

	static constexpr inline std::size_t max_code_points = 1;
	static constexpr inline std::size_t max_code_units  = 4;
	
	u8_encode_result encode_one(u32_span input, u8_span output,
		state& current, u8_encode_error_handler error_handler);
	u8_decode_result decode_one(u8_span input, u32_span output,
		state& current, u8_decode_error_handler error_handler);
};
```

Add more if you want speed, safety, etc.  
But 7 is the magic number.


### All Yours

- Each encoding object is its own type
- Strongly control its semantics and representation


### All Yours

- No Committee telling you what to do.
- No standard libraries saying its not important enough to be added.


### The Other Magical Bit Here?
