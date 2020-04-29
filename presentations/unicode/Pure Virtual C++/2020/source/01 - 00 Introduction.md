### Previous Work

Part 1 (Catch it [here](https://www.youtube.com/watch?v=BdUipluIf1E))  
Part 2 (Catch it [here](https://www.youtube.com/watch?v=FQHofyOgQtM))

<a href="https://www.youtube.com/watch?v=BdUipluIf1E"><img src="resources/Previously0.jpg" alt="Linked Picture of Previous YouTube Video on Unicode for C++23, Part 1" width="30%" height="30%" /></a> <a href="https://www.youtube.com/watch?v=FQHofyOgQtM"><img src="resources/Previously1.png" alt="Linked Picture of Previous YouTube Video on Unicode for C++23, Part 2" width="30%" height="30%" /></a>




# Our Constraints


### `char` is Bad

```cpp
#include <string>
#include <utility>

int main (int argc, char* argv[]) {
	std::string totally_utf8{argv[1]};
	my_utf8_function(std::move(totally_utf8)); // lol xd
}
```


#### `char` is Bad, Part 2

```cpp
#include <string>
#include <utility>

int main (int argc, char* argv[]) {
	std::string totally_utf8{argv[1]};
	my_utf8_function(std::move(totally_utf8)); // lol xd
}
```


### `wchar_t` is Bad

- UTF-16 on Windows
  - except when used with the Standard Library 🙃

- UTF-32 on POSIX-y machines
- ... Except if it's an IBM machine
  - UTF-16 on 32-bit machines
  - UTF-32 on 64-bit machines
  - None of the above in `zh-TW`, `jp-JP`, etc. locales


### `char16_t` and `char32_t` are bad

- `char16_t`: `#if defined(__STD_C_UTF_16__)` ➡ is UTF-16

- `char32_t`: `#if defined(__STD_C_UTF_32__)` ➡ is UTF-32

- otherwise: ???



### `char16_t` and `char32_t` ~~are Bad~~
### are Fine


Fixed in C++20 🎉!

[p1041 - make `char16`/`32_t` be UTF-16/32](https://wg21.link/p1041)

(Thanks, Robot 💚)


### Proposed fixes

- Get a new API that does not have the above issues?
  - (I am working on it)
- "Just make `char` UTF-8 and use UTF-8!"
  - "Just do it."
