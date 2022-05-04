# C++: Overcoming the Stockholm Syndrome

## Starting Point

What we want:

* fast, close to the hardware
* high-level abstractions, but still (close to) zero-cost

Only C++ can deliver this!

So we have to live with its idiosyncrasies and its horrendous unsafety ... right?


## What is C++?

Ingredients:

1. (most of) C
2. a whole lot of patches

Compatibility with C:

* main reason for its popularity
* main reason for its horribleness


## Some Horrible Features of C

* NULL pointers
  * [The Billion Dollar Mistake](https://www.infoq.com/presentations/Null-References-The-Billion-Dollar-Mistake-Tony-Hoare/)
* uninitialized variables, uninitialized fields in structs
  * [problem](https://godbolt.org/z/zxvasPo7E) (clang warns with `-Wall`), [solution](https://godbolt.org/z/cM3Wz1WYf)
* array decay, pointers without length
  * [problem](https://godbolt.org/z/78osf5aT5), [solution](https://godbolt.org/z/zMcbz87Eb)
* null-terminated strings
  * strings cannot contain the terminating character `\0`
  * it's easy to forget to provide space for the terminating character (`max_length + 1`)
  * [`strlen()`](https://en.cppreference.com/w/c/string/byte/strlen) has O(n) complexity
* [integer overflow](https://en.cppreference.com/w/cpp/language/operator_arithmetic#Overflows):
  unsigned wraps, signed causes UB
  * [problem](https://godbolt.org/z/8WTrhhxWs), [solution](https://godbolt.org/z/xKebGa7xh)
  * since C++20: unsigned [integer conversion](https://en.cppreference.com/w/cpp/language/implicit_conversion#Integral_conversions)
    is not *implementation-defined* anymore but defined as *modulo* behavior

C++ introduces some work-arounds, but it still provides all the footguns!


## Misguided Convenience (C)

* Implicit conversions
* Pointer arithmetic is too easy
* `if` without braces
  * ["goto fail" bug](https://nvd.nist.gov/vuln/detail/CVE-2014-1266)
  * Optional braces but required parens ([problem](https://godbolt.org/z/K8W9vnov7),
    GCC and clang show warning with `-Wall`)?
    Why not the reverse ([solution](https://godbolt.org/z/Wx6e5oYe6))?


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
  * [problem](https://godbolt.org/z/add17fMjv),
    [solution (mistake is recognized)](https://godbolt.org/z/Pq3EM9h5e),
    [mistake is fixed (nightly)](https://godbolt.org/z/ba79M9jjn),
    [mistake is fixed (stable)](https://godbolt.org/z/7eeWKha6h)
* no range checks by default (C++, because C has no checks at all)
  * `myvector[7]` vs. `myvector.at(7)`


## Half-Assed Improvements in C++

* `nullptr` instead of `NULL`
* references
  * [problem](https://godbolt.org/z/Yh3KfbPro) (warning is issued),
    [solution](https://godbolt.org/z/KjK519v5j)
  * [problem](https://godbolt.org/z/oEeM4o5qo) (no warning),
    [solution](https://godbolt.org/z/bK36rvqh6)
* `std::optional`
  * [problem](https://godbolt.org/z/GoGnb7Tce), [solution](https://godbolt.org/z/oc9ePP853)
  * [problem (memory overhead)](https://godbolt.org/z/G4va5nMYh),
    [solution (niche optimization)](https://godbolt.org/z/h8455oqrh)
* `std::string`
  * an empty string must still store a terminating null character for
    [`c_str()`](https://en.cppreference.com/w/cpp/string/basic_string/c_str)
* move semantics
* smart pointers
* `std::string_view`, `std::span`
* `enum class`
* `std::variant`


## Unwieldy Library Types That Should Be Built-Ins

* `std::array`, `std::tuple`
* `std::string_view`, `std::span`
* `std::variant`, `visit()`?


## Nice Things That We Cannot Have Due to Backwards Compatibility

* exhaustive `switch`
  * [problem](https://godbolt.org/z/7rKW1fE4K)
    (clang produces a warning by default, GCC with `-Wall`),
    [solution](https://godbolt.org/z/7oE9YjG3M)


## C++ Hall of Shame

* `auto_ptr`
* `iostream`: ridiculously slow, awkward error handling
* preprocessor
