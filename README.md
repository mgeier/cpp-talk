# C++: Overcoming the Stockholm Syndrome

## Starting Point

What we want:

* fast, close to the hardware
* high-level abstractions, but still (close to) zero-cost

Only C++ can deliver this!

So we have to live with its idiosyncrasies ... right?


## What is C++?

* most of C
* a whole lot of patches


## Some Horrible Features of C

* NULL pointers
  * [The Billion Dollar Mistake](https://www.infoq.com/presentations/Null-References-The-Billion-Dollar-Mistake-Tony-Hoare/)
* uninitialized variables, uninitialized fields in structs
* array decay, pointers without length
  * [problem](https://godbolt.org/z/KGqa6rhMe), [solution](https://godbolt.org/z/P8zjKzY9q)
* integer overflow: unsigned wraps, signed causes UB

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
* thread un-safe by default
  * and never *really* thread-safe if you're honest
  * [ThreadSanitizer](https://clang.llvm.org/docs/ThreadSanitizer.html) helps
  * [problem](https://godbolt.org/z/q87dqfhYe),
    [solution (mistake is recognized)](https://godbolt.org/z/rhj44e1fP),
    [mistake is fixed (nightly)](https://godbolt.org/z/oTr44sfej),
    [mistake is fixed (stable)](https://godbolt.org/z/v1o5Gxv9M)
* no range checks by default (C++, because C has no checks at all)


## Half-Assed Improvements in C++

* `nullptr` instead of `NULL`
* references
  * [problem](https://godbolt.org/z/ssMKx1874) (warning is issued),
    [solution](https://godbolt.org/z/scboa6YqY)
  * [problem](https://godbolt.org/z/es9dqxnWv) (no warning),
    [solution](https://godbolt.org/z/qvq7r7WKW)
* move semantics
* enum class
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
