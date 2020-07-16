# Consider `any_encoding`


<a href="https://youtu.be/FQHofyOgQtM?t=1561"><img src="resources/any_encoding.png" alt="Linked Picture of Previous YouTube Video on Unicode for C++ in Greater Detail, Part 2" width="72%" height="72%" /></a>


### `std::function`, but for Encodings

Catch it in [Part 2](https://youtu.be/FQHofyOgQtM?t=1561):

- constructs from an encoding
- `decode_one` takes bytes, outputs code points
- `encode_one` takes code points, outputs bytes
- gratuitously oversized `max_code_points`/`max_code_units`


### `any_encoding`, internals

type-erases the encoding and stores it inside of itself
- on `enc.decode_one`, unwraps `this` and calls real `stored_encoding.decode_one`
- on `enc.encode_one`, unwraps `this` and calls real `stored_encoding.encode_one`


### Consider basis loop

Think about `decode_one` for `any_encoding` with 1,000,000 code points.


### Consider basis loop

- Loop (`* 1'000'000`)
  - Get to call (OVERHEAD)
  - Unwrap, do real call
  - Exit call (OVERHEAD)

`(2 ✖ (OVERHEAD)) ✖ 1'000'000`


# 😬


### Can do better!

Check if `text_decode` function exists:

- If it does, call that instead of running `enc.decode_one` in a loop
  - Unwrap
  - Do 1,000,000 code points without overhead
  - Exit function


# Much Nicer Flow

### Much Better Performance 👍


### All The Extensions!

Write any friend / free function like so:

- `text_transcode_one` (helps with ranges)
- `text_decode`/`text_encode`
- `text_transcode`
- `text_decode_count`/`text_encode_count`
- `text_validate`/`text_validate_as`


### The library calls them for you

A one-to-one relationship; always prefers your implementation over internals

- core loop/ranges ➡ `text_transcode_one`
- `decode`/`encode` ➡ `text_decode`/`text_encode`
- `transcode` ➡ `text_transcode`
- `decode_count`/`encode_count` ➡ `text_decode_count`/`text_encode_count`
- `validate`/`validate_as` ➡ `text_validate`/`text_validate_as`


### Everyone Speaks the same language

```cpp
class session {
	// ... init with proper encoding earlier...
	txt::any_encoding from_encoding_;
	std::vector<std::byte> res_body_;
	// ...

	void session::on_read(beast::error_code ec, std::size_t bytes_transferred) {
		/* … skip boilerplate */
		std::span<std::byte> input_bytes(this->res_body_.data(), bytes_transferred);
		std::ranges::unbounded_view output(
			std::back_inserter(this->converted_body_)
		);

		txt::utf8 to_encoding{};

		// This will call YOUR extension points, if they are there!!
		txt::transcode(input_bytes, output, this->from_encoding_, to_encoding);

		std::clog << this->converted_body_ << std::endl;
};
```


### Yours, or Someone Else's

Open ecosystem!

- No longer tied to (proprietarily controlled) standard library
- Someone can make a bucket of encodings they want
  - e.g.: WHATWG has dozens of encodings for e-mail interchange
  - Can make them fast for encoding pairs you care about, once
  - EVERYONE benefits


### Usages?

[Fast validation with Daniel Lemire](https://lemire.me/blog/2018/10/19/validating-utf-8-bytes-using-only-0-45-cycles-per-byte-avx-edition/)

<a href="https://lemire.me/blog/2018/10/19/validating-utf-8-bytes-using-only-0-45-cycles-per-byte-avx-edition/"><img src="resources/Lemire.png" alt="Linked Picture of Daniel Lemire's blog post on Fast UTF-8 Validation" width="55%" height="55%" /></a>


### Usages?

[Fast UTF-8 Conversion with DFAs](https://www.youtube.com/watch?v=5FQ87-Ecb-A)

<a href="https://www.youtube.com/watch?v=5FQ87-Ecb-A"><img src="resources/Steagall.jpg" alt="Linked Picture of Bob Stegall's CppCon 2019 Fast UTF-8 Conversion talk" width="65%" height="65%" /></a>


### Usages?

Platform specific APIs

<a href="https://developer.apple.com/documentation/coreservices/1433517-convertfromtexttounicode"><img src="resources/doc-apple-ConvertFromTextToUnicode.png" alt="Linked Picture of Apple Conversion Function Documentation" width="30%" height="30%" /></a>
<a href="https://docs.microsoft.com/en-us/windows/win32/api/stringapiset/nf-stringapiset-multibytetowidechar"><img src="resources/doc-win-multibytetowidechar.png" alt="Linked Picture of Windows Conversion Function Documentation" width="30%" height="30%" /></a>
<a href="https://www.gnu.org/savannah-checkouts/gnu/libiconv/documentation/libiconv-1.15/iconv.3.html"><img src="resources/doc-iconv.png" alt="Linked Picture of iconv conversion function" width="30%" height="30%" /></a>


### And More
