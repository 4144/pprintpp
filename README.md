# Python style printf for C++

----
## What is `pprintpp`?

The acronym stands for "Python style print for C plus plus".

`pprintpp` is a **header-only** C++ library, which aims to make `printf` use safe and easy.
It is a *pure compile time library*, and will add **no overhead** to the runtime of your programs.


## Dependencies

The library does only depend on **C++11** (or higher) and the **STL** (`<tuple>` and `<type_traits>`).

The STL dependency can easily get rid of by reimplementing some type traits. This way `pprintpp` can be used in hardcore baremetal environments (where it already has been in use actually).

## Printf Compatibility

`pprintpp` will transform a tuple `(format_str, [type list])` to `printf_compatible_format_string`.

That means, that it can be used with any `printf`-like function. You just need to define a macro, like for example these ones for `printf` and `snprintf`:

``` c++
#define pprintf(fmtstr, ...) printf(AUTOFORMAT(fmtstr, ## __VA_ARGS), ## __VA_ARGS__)
#define psnprintf(outbuf, len, fmtstr, ...) \
    snprintf(outbuf, len, AUTOFORMAT(fmtstr, ## __VA_ARGS__), ## __VA_ARGS__)
```

Embedded projects, which introduce their own logging/tracing functions, which accept `printf`-style format string, will also profit from this library.

## Example

When using `printf`, the programmer has to choose the right types in the format string.
``` C++
printf("An int %d, a float %f, a string %s\n", 123, 7.89, "abc");
```
If this format string is wrong, the programm will misformat something in the best case.
In the worst case, the program might even crash.

The python style print library allows for the following:
``` C++
pprintf("An int {}, a float {}, a string {s}\n", 123, 7.89, "abc");
```
The types are chosen *automatically* **at compile time**.
This is both safe *and* convenient.


The assembly generated from the simple program...
``` c++
int main()
{
    pprintf("{} hello {s}! {}\n", 1, "world", 2);
}
```

...shows, that this library comes with *no* runtime overhead:
```
bash $ objdump -d example
...
0000000000400450 <main>:
  400450:       48 83 ec 08             sub    $0x8,%rsp
  400454:       41 b8 02 00 00 00       mov    $0x2,%r8d
  40045a:       b9 04 06 40 00          mov    $0x400604,%ecx # <-- "world"
  40045f:       ba 01 00 00 00          mov    $0x1,%edx
  400464:       be 10 06 40 00          mov    $0x400610,%esi # <-- "%d hello world %s!..."
  400469:       bf 01 00 00 00          mov    $0x1,%edi
  40046e:       31 c0                   xor    %eax,%eax
  400470:       e8 bb ff ff ff          callq  400430 <__printf_chk@plt>
  400475:       31 c0                   xor    %eax,%eax
  400477:       48 83 c4 08             add    $0x8,%rsp
  40047b:       c3                      retq
...
```

Dumping the read-only data section of the binary shows the `printf` format string.
It looks as if the programmer had directly written the printf line without ever having used `pprintpp`:
```
bash $ objdump -s -j .rodata example
...
Contents of section .rodata:
 400600 01000200 776f726c 64000000 00000000  ....world.......
 400610 25642068 656c6c6f 20257321 2025640a  %d hello %s! %d.
 400620 00                                   .
```
