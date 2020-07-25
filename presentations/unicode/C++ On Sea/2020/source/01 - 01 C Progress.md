# Wait...

ThePhD, you wrote a C paper?


### N2440

![From N2440 - Current Function Definitions for transcoding between types shown as a matrix.](resources/C%20-%20current%20functions.png)


### N2440 - Fixes

![From N2440 - Current Function Definitions for transcoding between types shown as a matrix with proposed functions there.](resources/C%20-%20proposed%20functions.png)


### N2440 Passes Initial Pitch!

Approved Ithaca, New York - October 2019

- Mailed musl-libc and glibc to get started
- Implemented it privately as well

And yet...




# Something... Off


### An Example

```cpp
size_t mbrtowc(
	wchar_t* ws,
	const char* s, size_t n,
	mbstate_t* state
);
```


### Given

We are working with:

- `char` "execution" locale encoding: Big5-HKSCS
	- `sizeof(char)` == 8 bit/1 byte

- `wchar_t` "wide execution" locale encoding: UTF-32
	- `sizeof(wchar_t)` == 32-bit/4 bytes


### An Example

```cpp
size_t mbrtowc(
	wchar_t* ws,
	const char* s, size_t n,
	mbstate_t* state
);
```



### An Example - Usage

```cpp
int main (int argc, char* argv[]) {
	wchar_t wide_output[4];
	const char input[] = "\x88\x62\x58\x58\x58";
	mbstate_t state{};

	size_t r = mbrtowc(wide_output, input, 3, &state);

	switch (r) {
		case (size_t)0: /* null character found */
		case (size_t)-1: /* invalid sequence */
		case (size_t)-2: /* incomplete sequence */
		default: /* ... */
	}

	return 0;
}
```


### We're Lying

About a lot of things...


### Lie 1: Multiple characters

```cpp
size_t mbrtowc(
	wchar_t* pwc, // pointer to SINGLE wide character
	const char* s, size_t n,
	mbstate_t* state
);
```

```cpp
size_t wcrtomb(
	char* s, // pointer to MULTIPLE characters
	wchar_t wc,
	mbstate_t* state
);
```


### Lie 2: state can be brace-zeroed

```cpp
int main (int argc, char* argv[]) {
	wchar_t wide_output[4];
	const char input[] = "\x88\x62\x58\x58\x58";

	mbstate_t state;
	// init to initial state
	size_t init = mbrtowc(NULL, "", 1, &state);
	assert(init == 0);

	size_t r = mbrtowc(wide_output, input, 3, &state);

	/* ... */
	return 0;
}
```


### Lie 3: Implementations respect the standard

```cpp
int main (int argc, char* argv[]) {
	wchar_t wide_output[4];
	const char input[] = "\x88\x62\x58\x58\x58";
	mbstate_t state;

	size_t r = mbrtowc(wide_output, input, 3, &state);

	switch (r) {
		case (size_t)0: /* null character found */
		case (size_t)-1: /* invalid sequence */
		case (size_t)-2: /* incomplete sequence */
		default: /* ... */
	}

	return 0;
}
```


### Lie 3: Implementations respect the standard

```cpp
int main (int argc, char* argv[]) {
	wchar_t wide_output[4];
	const char input[] = "\x88\x62\x58\x58\x58";
	mbstate_t state{};

	size_t r0 = mbrtowc(wide_output, input, 1, &state);
	assert(r0 == -2);
	size_t r1 = mbrtowc(wide_output, input + 1, 1, &state);
	assert(r1 == 2); // ???
	size_t r2 = mbrtowc(wide_output + 1, input + 2, 1, &state);
	assert(r2 == 0);

	return 0;
}
```


### Unfixable??

"Putting out two wide characters is not defined by the standard, so anything can be done here."

"...  Big5 is simply not an encoding supported by ISO C..."


### ⁉

How is this portable behavior...?

How is this reliable behavior...?
