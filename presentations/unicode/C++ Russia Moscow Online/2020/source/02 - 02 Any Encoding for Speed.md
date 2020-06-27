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

Check if `text_decode` function exists on object:

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


### Usages?

[Fast validation with Daniel Lemire](https://lemire.me/blog/2018/10/19/validating-utf-8-bytes-using-only-0-45-cycles-per-byte-avx-edition/)

<a href="https://youtu.be/FQHofyOgQtM?t=1561"><img src="resources/daniel-lemire.png" alt="Linked Picture of Daniel Lemire's blog post on Fast UTF-8 Validation" width="55%" height="55%" /></a>
