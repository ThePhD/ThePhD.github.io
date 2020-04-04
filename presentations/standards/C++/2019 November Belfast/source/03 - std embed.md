### `std::embed(...)`

- Magic `consteval` function backed by compiler-blessed built-in
- Takes a `std::string_view` representing "a resource"
- Implementation defined lookup and interpretation of string


### Returns: `std::span<const std::byte>`

- View into compiler-backed static memory.
- Explicitly contains `.data()` and `.size()`
- No null termination or otherwise: just the bytes.


### `std::embed_str` ?

- Not proposed, others can provide null termination
  - e.g. `constexpr std::string` using iterator-pair constructor
- Not opposed, just trying to do the simple thing first here


### Arbitrary Compile-Time Read and Compute

Link: [Live Example!](https://godbolt.org/z/E7hmiE)


### `std::embed` Tradeoffs

- ✔️ Works at constant evaluation time (right on time!)
- ✔️ Implementable today (built-ins work just fine)
- ✔️ Enables complex use cases not at all covered by non-programmable alternative
- ❌ Must copy out of span to separate storage to apply attributes.
- ❌ Not discoverable by ancient build tech (requires user to list (glob) dependency)
