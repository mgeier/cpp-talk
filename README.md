# C++: Overcoming the Stockholm Syndrome

## Starting Point

What we want:

* fast, close to the hardware
* high-level abstractions, but still (close to) zero-cost

Only C++ can deliver this!

So we have to live with its idiosyncrasies and its horrendous unsafety ... right?


## What is C++?

* most of C
* a whole lot of patches


## Some Horrible Features of C

* NULL pointers
  * [The Billion Dollar Mistake](https://www.infoq.com/presentations/Null-References-The-Billion-Dollar-Mistake-Tony-Hoare/)
* uninitialized variables, uninitialized fields in structs
  * [problem](https://godbolt.org/z/K5xdE6ddd), [solution](https://godbolt.org/z/3rKzz4Pqb)
* array decay, pointers without length
  * [problem](https://godbolt.org/z/KGqa6rhMe), [solution](https://godbolt.org/z/P8zjKzY9q)
* null-terminated strings
  * strings cannot contain the terminating character `\0`
  * it's easy to forget to provide space for the terminating character (`max_length + 1`)
  * [`strlen()`](https://en.cppreference.com/w/c/string/byte/strlen) has O(n) complexity
* [integer overflow](https://en.cppreference.com/w/cpp/language/operator_arithmetic#Overflows):
  unsigned wraps, signed causes UB
  * [problem](https://godbolt.org/z/P8bWb4PWb), [solution](https://godbolt.org/z/hvYMMc836)
  * since C++20: unsigned [integer conversion](https://en.cppreference.com/w/cpp/language/implicit_conversion#Integral_conversions)
    is not *implementation-defined* anymore but defined as *modulo* behavior

C++ introduces some work-arounds, but it still provides all the footguns!


## Misguided Convenience (C)

* Implicit conversions
* Pointer arithmetic is too easy
* `if` without braces
  * ["goto fail" bug](https://nvd.nist.gov/vuln/detail/CVE-2014-1266)
  * Optional braces but required parens ([problem](https://godbolt.org/z/nzzY313EY))?
    Why not the reverse ([solution](https://godbolt.org/z/MW3Ts3x7r))?


## Misguided Convenience (C++)

* implicit `this->`
* ...?


## Wrong Defaults (C and C++)

* mutable by default
* deep copy by default (C++)
  * moving would be more efficient
* thread un-safe by default
  * and never *really* thread-safe if you're honest
  * [ThreadSanitizer](https://clang.llvm.org/docs/ThreadSanitizer.html) helps
  * [problem](https://godbolt.org/z/q87dqfhYe),
    [solution (mistake is recognized)](https://godbolt.org/z/rhj44e1fP),
    [mistake is fixed (nightly)](https://godbolt.org/z/oTr44sfej),
    [mistake is fixed (stable)](https://godbolt.org/z/v1o5Gxv9M)
* no range checks by default (C++, because C has no checks at all)
  * `myvector[7]` vs. `myvector.at(7)`


## Half-Assed Improvements in C++

* `nullptr` instead of `NULL`
* references
  * [problem](https://godbolt.org/z/ssMKx1874) (warning is issued),
    [solution](https://godbolt.org/z/scboa6YqY)
  * [problem](https://godbolt.org/z/es9dqxnWv) (no warning),
    [solution](https://godbolt.org/z/qvq7r7WKW)
* `std::string`
  * an empty string must still store a terminating null character for
    [`c_str()`](https://en.cppreference.com/w/cpp/string/basic_string/c_str)
* move semantics
* `enum class`
* smart pointers
* `std::optional`/`std::variant`


## Unwieldy Library Types That Should Be Built-Ins

* `std::array`, `std::tuple`
* `std::string_view`, `std::span`
* `std::variant`, `visit()`?


## Misc

* non-exhaustive? (some C++ compilers issue warnings)


## C++ Hall of Shame

* `auto_ptr`
* `iostream`: ridiculously slow, awkward error handling
* preprocessor
