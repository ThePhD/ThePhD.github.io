# Is... Is it Fast?


### 🐌

Nope.



### Basis Formula is Correct, but...?

for text `input` and writable `output` with objects `α` and `β`:

- if `input` is empty, return. otherwise: (⚠!!)
  - call `α.decode_one` on `input` into `span` over  
  `code_point intermediate[max_code_points]`; (⚠!!)
  - if error occurred, bail with error;
  - call `β.encode_one` with `intermediate` into `output`;
  - if error occurred, bail with error;
  - update `input` to move forward by # of `code unit`s read;


### Inefficiency?

Intermediate not always necessary:

- UTF-EBCDIC ⟺ UTF-8 ("UTF-8 mod" step)
- UTF-16 ⟺ UTF-32 (non-surrogates directly writable)
- ASCII/Latin-1 ⟺ UTF-8 (compatible)
- ASCII/GBK encodings ⟺ GB18030 (compatible)


### In other words

Why do:

α `code_unit`s ⟺ `code_point`s ⟺ β `code_unit`s

instead of:

α `code_unit`s ⟺ β `code_unit`s

where possible?


### More Problems - Single vs. Bulk

> On musl (where I'm familiar with performance properties),
> byte-at-a-time conversion is roughly half the speed of bulk...

– [Rich Felker, Monday December 30th, musl-libc mailing list](https://www.openwall.com/lists/musl/2019/12/30/8)


### Single vs. Bulk

Consider C Standard function `mbrtowc`;  
converts one block of `char*` to a `wchar_t`:

- maybe function in shared object / dynamic link library
- indirect calls through locale tables
- no SIMD on non-bulk data

Also...


#### Invariant Checking, `mbrtowc`

![Is This Null Butterfly Meme](resources/ButterflyMeme.jpg)


#### Invariant Checking, `mbsrtowcs`

![Is This Null Butterfly Meme, as a pattern](resources/ButterflyMemePattern.jpg)


### We Can Do Better

Take advantage of...

- shortcut round tripping without common code point
- take advantage of additional information on encoding object
- [`T*, T*`) or [`T*, size`) ranges to do SIMD / fast computation
- checking invariants once, not `std::size(input)` times


### We Need Speed Extensions
