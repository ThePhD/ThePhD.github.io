# Okay, But...

Is it fast?


### Yes

Better on both memory and speed than every other kind of tool, no matter the kind of data (embedding large neural networks and targeting embedded compilers, etc.).

Data: https://github.com/ThePhD/embed#speed-results


### Speed Matters

> gcc wants 12+ GB to build this (it's just that one 60 MB array). Using `uint64_t` instead of `uint8_t` significantly reduces gcc memory usage, but for some reason compile time explodes. - [Source](https://twitter.com/oe1cxw/status/1008361214018244608)

> Then again, it would probably be wiser to put that energy in `std::embed`, which does beat all the optimizations presented here by orders of magnitude! - [Source](https://cor3ntin.github.io/posts/arrays/)


### Cross-platform Matters

> Apparently this (link) is a thing...  
> \> _fatal error C1091: compiler limit: string exceeds 65535 bytes in length_  
> So maybe another time... – [Source](http://lists.llvm.org/pipermail/llvm-dev/2020-January/138310.html)

> `// generated content broken into pieces because MSVC is in the 1990s.` - [Source](https://github.com/libnonius/nonius/blob/devel/include/nonius/reporters/html_reporter.h++#L42)


### Implementer Time Matters

This is not just Quality of Implementation.

- GCC bug has been open for 16 years about this
- LLVM bugs indicate no progress will be made (someone tried to)
- Newly developed compilers still take exponential compiler size explosion (tokenization)
