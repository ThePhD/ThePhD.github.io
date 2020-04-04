### `#embed`/`#embed_str`


- Preprocessor directive
- Does not exist past Translation Phase 4.
- string-literal token using "resource" names like `<foo>` or `"foo"`
- Implementation-defined lookup


### `#embed`

- Behaves "as if" it creates `{ integer, integers... }` comma-separated list of the contents
  - Need "as if" to allow for magical internal syntax by advanced implementations
- Can be used to initialize anything that can initialize from a brace-delimited list


### `#embed`

```cpp
const char char_array[] =
    #embed "data.bin"
; // regular ol' char array

constexpr const unsigned char uchar_array[] =
    #embed "art.png"
; // initialize constexpr data too

const std::vector<unsigned> vec =
    #embed "data.bin"
; // initialized as list of unsigned ints from each byte
```


### `#embed_str`

- Produces a string-literal of the contents
- Satisfies null termination expectation for many ""text"" files


### `#embed_str`

```cpp
const char data_str[] =
    #embed_str "shader.txt"
; // guaranteed null termination

const char aligned_data_str[] __attribute__ ((aligned (8))) =
    #embed "natural_text.xml"
; // attributes work fine too
```


### `#embed` Tradeoffs

- ✔️ Immediately discoverable by existing tools (works with `-(M)MD`)
- ✔️ Implementable today with no change to existing compiler infrastructure
- ✔️ Explicit layout control (attributes and more) for data variable
- ✔️ Simplicity (is a feature)
- ❌ Entirely static, non-programmable; "too early" to do anything beyond basics
