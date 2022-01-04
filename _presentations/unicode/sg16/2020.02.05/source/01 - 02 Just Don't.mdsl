— A bunch of people at Committee Meetings, and everyone who didn't fully read [utf8-everywhere](https://utf8everywhere.org/)


# Hot Take: Just Don't


### A mil×sÖ»¯¤± and one 🐞

- "Why does my `std::string` contain garbage...?"
- `char` for system encoding AND UTF-8 is **wrong**
- `char` will never be UTF-8, always
  - "I can enforce it in my codebase!"


### Game Over.

```cpp
int main(int argc, char* argv[]) {
                   // ^^ you lose.
	return 0;
}
```


### You Are Not 
### Google/Bloomberg/Facebook
### /Microsoft/{BigTech™}

- You don't own the entire tech stack.
- You don't own the user's locale encoding.
- You are a piece in a much bigger 🥧.


# Don't forget it.


### C++20 Safety: `char8_t`

- `std::u8string` ≠ `std::string`
	- No implicit conversions
	- No accidental serializations
	- Compiler errors when you forget
	- Must escape intentional uses
	- `char8_t` can't alias: better code generation


### C++ < 20 Safety: `unsigned char`

- `std::basic_string<unsigned char>` ≠ `std::string`
	- No implicit conversions
	- No accidental serializations
	- Compiler errors when you forget
	- Must escape intentional uses
	- `unsigned char` can still alias (worse code generation)
