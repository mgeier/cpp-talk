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

C++ introduces some work-arounds, but it still provides all the footguns!


## Misguided Convenience (C)

* Implicit conversions
* Pointer arithmetic is too easy
* `if` without braces
  * ["goto fail" bug](https://nvd.nist.gov/vuln/detail/CVE-2014-1266)
  * Optional braces but required parens? Why not reverse?


## Misguided Convenience (C++)

* implicit `this->`


## Wrong Defaults (C and C++)

* mutable by default
* deep copy by default
* thread un-safe by default
  * and never *really* thread-safe if you're honest
  * ThreadSanitizer helps
  * problem: https://godbolt.org/z/MjobjEao6
  * solution:
    * https://godbolt.org/z/76anTWodP
    * https://godbolt.org/z/znTc7azec (with atomic counter)
* no range checks by default


## Half-Assed Improvements in C++

* nullptr
* references
  * [problem](https://godbolt.org/z/ssMKx1874) (warning is issued),
    [solution](https://godbolt.org/z/scboa6YqY)
  * [problem](https://godbolt.org/z/es9dqxnWv) (no warning),
    [solution](https://godbolt.org/z/qvq7r7WKW)
* move semantics
* enum class
* smart pointers
* optional/variant


## unwieldy library types that should be built-ins

* std::array, std::tuple
* std::string_view, std::span
* std::variant, visit()?


## misc

* non-exhaustive? (some C++ compilers issue warnings)
* overflow: unsigned wraps, signed causes UB


## C++ Hall of Shame

* `auto_ptr`
* `iostream`: ridiculously slow, awkward error handling
* preprocessor
