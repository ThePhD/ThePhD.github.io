# The Final Problem


![Picture of the Encoding Loop from the YouTube Unicode for C++23 Video CppCon 2019 Presentation](resources/SimpleIdea.jpg)


### Connecting the 🌍

`to`/`from` combinations are unique and pairwise
- quadratic number of registrations to cover all possible encoding combinations
- 50 encodings? Need 2500 registrations for complete interop
- `iconv` has over 48 encodings - has not brought the world down yet


### 🙏 Pouring out a Blessing 🙏

If you transcode from your encoding to a blessed:
- UTF-32, UTF-16, UTF-8; or
- a surrogate-allowing, "unchecked" version of the above

Registry automatically "connects" to other encodings through intermediates above


### Connected Registries

```cpp
int main (int argc, char* argv[]) {
	conversion_registry* reg;
	/* ... */
	// connect encodings
	add_to_registry(reg, "𝔓𝔥𝔣𝔱𝔞𝔤𝔫-72", "UTF-32",
		&𝔓𝔥𝔣𝔱𝔞𝔤𝔫72s_to_u32s, NULL, NULL);
	add_to_registry(reg, "UTF-32", "utf8",
		&u32s_to_u8s, NULL, NULL);

	conversion* conv;
	open_error r = conversion_open(&conv, "𝔓𝔥𝔣𝔱𝔞𝔤𝔫-72", "utf-8");
	assert(r == OPEN_ERR_OKAY); // works!!
	
	/* ... */
	return 0;
}
```


### Once more...

We are freed from the Standard

- no longer glibc's job to give X encoding
- no longer musl-libc's job to give Y encoding
- give fast pairwise encodings when you want
- let the fallback handle it if a path through UTF is available




# My use case matters

Screw your market share.

We have our users. We have our needs.

Both me, and,
