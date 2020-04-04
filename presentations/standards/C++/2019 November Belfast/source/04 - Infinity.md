### Embedders: Infinity Wars

```cpp
const char random_str[] =
    #embed_str "/dev/urandom"
;

constexpr auto random_bytes = std::embed("/dev/urandom");
```

Uh oh.


### Network Drives, char Devices, and Mapped files...

- Oh my!
- Ban-list on implementations? (Poor Quality of Implementation!)
- Weasel wording in the standard? (Hard to specify!)


![FF7 Cloud Limit Break](resources/Limit.jpg)


### <span style="color:darkred">L</span><span style="color:yellowgreen">i</span><span style="color:darkgreen">m</span><span style="color:blue">i</span><span style="color:violet">t</span> on Embed Directives

- "Maximum allowable" number of elements in the string/array/etc.
- Actual size of the resource can be lower.

```cpp
const char random_str[] =
    #embed_str 32 "/dev/urandom"
;

constexpr std::span<const std::byte> random_bytes = 
    std::embed("/dev/urandom", 32);
```
