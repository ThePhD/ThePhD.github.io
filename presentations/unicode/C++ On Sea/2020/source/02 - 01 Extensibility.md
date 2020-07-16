# The Richness of Life


### Previous Interfaces

- prioritized local maxima
- gave outsize control to implementation over end-user


### Where is {INSERT HERE} Encoding??

🤷‍♀️


### User-defined Encodings?

`iconv` - comes preconfigured, have to hack the source  
`encoding_rs` - match concepts to build encoding  
`Boost.Text` - No support; must convert on your own  
`text_view` - extensible, no transcoding support out of the box  
`libogonek` - extensible, provides transcoding/normalization support  


### C Standard: Just Make a Locale

"Just" 🙃


### However, Notice...

```cpp
conv_error XsrtoYs(
	size_t* p_output_size, YC** p_output,
	size_t* p_input_size, const XC** p_input,
	mcstate_t* state
);
```

```cpp
size_t iconv (iconv_t cd,
	const char** inbuf, size_t* inbytesleft,
	char** outbuf, size_t* outbytesleft
);
```


### Hmm... 🤔

What if... we actually learned something from 19 years of iconv...?


### Haha!

Just kidding,,,

- Can't just hide type of `XC*`/`YC*` behind an `unsigned char*`
- Can't just store conversion state in some type...


### Unless... 😳

```cpp
conv_error convert(conversion* conv,
	size_t* p_output_size, unsigned char** p_output,
	size_t* p_input_size, const unsigned char** p_input
);
```


### 🥰 Amazing 🥰

- Type erased encodings
- "descriptor" type `conversion` tells us what to do
- Bulk, keeps track of size, gives us error


# [[segue]]

When RSize Matters


### Remember Safe C Library and Annex K?

![Picture of RSIZE_MAX-style constants in Safe C Lib](resources/safeclib-doc.png)

"Who needs to work with more than 4 KB at a time?"


### Resulted in soft DoS vulnerabilities

"Boy, these safe libs are great, `RSIZE_MAX` is definitely supposed to be `SIZE_MAX >> 1` like the C Standard Recommends! Safety AND space!"


### My Face When Bug Reports on Certain Platforms

🤪?!
🤬!!!
😬....
🤡.


### Annex K Sucks

> "FOH with this crap got-damn!!" - [Martin Sebor, September 30th 2015, N1969](www.open-std.org/jtc1/sc22/wg14/www/docs/n1969.htm#tc). Probably.



# [[end_segue]]




### Alas 😫

`iconv`'s model only accommodates descriptors pulled from a fixed pool
- Everything programmed in statically
- (mostly; some allocations here and there)
