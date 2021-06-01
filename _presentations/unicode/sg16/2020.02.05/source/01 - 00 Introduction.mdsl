# State of Things



### Previous Work

Catch it [here](https://www.youtube.com/watch?v=BdUipluIf1E)

<a href="https://www.youtube.com/watch?v=BdUipluIf1E"><img src="resources/Previously.jpg" alt="Linked Picture of Previous YouTube Video on Unicode for C++23" width="60%" height="60%" /></a>


### Cannot handle Multibyte Encodings

"What's a Unicode?"

```cpp
int isblank( int ch );
int tolower( int ch );
int toupper( int ch );
// says "int", does "unsigned char"
```




### `char` sucks

```cpp
#include <string>
#include <utility>

int main (int argc, char* argv[]) {
	std::string totally_utf8{argv[1]};
	my_utf8_function(std::move(totally_utf8)); // lol
}
```


### `wchar_t` sucks

- UTF-16 on Windows (except when used with the Standard Library?!)

- UTF-32 on POSIX-y machines
- ... Except if it's an IBM machine
  - UTF-16 on 32-bit machines
  - UTF-32 on 64-bit machines
  - None of the above in `zh-TW`, `jp-JP`, etc. locales


### `char16_t` and `char32_t` suck

- `char16_t`: `#if defined(__STD_C_UTF_16__)` ➡ is UTF-16

- `char32_t`: `#if defined(__STD_C_UTF_32__)` ➡ is UTF-32

- otherwise: ???



### `char16_t` and `char32_t` ~~suck~~
### are fine


Fixed in C++20 🎉!

[p1041 - make `char16`/`32_t` be UTF-16/32](https://wg21.link/p1041)

(Thanks, Robot ♥)




### Lossy Operations

`std::wctomb` -> `std::mbtoc32` -> Code Point  
Code Point -> `std::c32tomb` -> `std::mbtowc`

Works if and only if your locale is UTF-8 ... 😫


### "Just make `char` UTF-8 and use UTF-8!"

"Just do it."
