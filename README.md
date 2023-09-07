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
  * [problem](https://godbolt.org/z/zxvasPo7E) (clang warns with `-Wall`),
    [solution](https://godbolt.org/z/b5MPM94P7),
    [Miri detects undefined behavior in unsafe code](https://play.rust-lang.org/?version=stable&edition=2021&gist=cedaffcf2203b20533f6ebdaf9db35ed)
  * if uninitialized data is really needed:
    [no overhead](https://godbolt.org/z/s87M35fjM), [still no overhead](https://godbolt.org/z/5hbEhW56n)
* array decay, pointers without length
  * [problem](https://godbolt.org/z/78osf5aT5), [solution](https://godbolt.org/z/zMcbz87Eb)
* null-terminated strings
  * strings cannot contain the terminating character `\0`
  * it's easy to forget to provide space for the terminating character (`max_length + 1`)
  * [`strlen()`](https://en.cppreference.com/w/c/string/byte/strlen) has O(n) complexity
* [integer overflow](https://en.cppreference.com/w/cpp/language/operator_arithmetic#Overflows):
  unsigned wraps, signed causes UB
  * [problem](https://godbolt.org/z/8WTrhhxWs), [solution](https://godbolt.org/z/oa64cWdqx)
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
* values and references are indistinguishable at the call site
  * `may_or_may_not_mutate(value)`: the `value` might be copied/moved or passed by reference,
    we can only find out by looking at the function signature.


## Wrong Defaults (C and C++)

* mutable by default
* deep copy by default (C++)
  * moving would be more efficient
* thread un-safe by default
  * and never *really* thread-safe if you're honest
  * [ThreadSanitizer](https://clang.llvm.org/docs/ThreadSanitizer.html) helps
  * [problem (ThreadSanitizer complains)](https://godbolt.org/z/Gn9TExKqx),
    [solution (mistake is recognized by the compiler)](https://godbolt.org/z/oq8YfoG4G),
    [mistake is fixed (using `std::thread::scope`)](https://godbolt.org/z/as4EKcPWq),
    [mistake is fixed (using `std::thread::spawn`)](https://godbolt.org/z/7eeWKha6h)
* no range checks by default (C++, because C has no checks at all)
  * `myvector[7]` vs. `myvector.at(7)`


## Half-Assed Improvements in C++

* `nullptr` instead of `NULL`
* references
  * [problem](https://godbolt.org/z/v73aGeGKx) (warning is issued),
    [solution](https://godbolt.org/z/njKM7x3sT)
  * [problem](https://godbolt.org/z/nherE5h5z) (no warning, but AddressSanitizer complains),
    [improvement](https://godbolt.org/z/EG1Tvoa4r) (warning with the experimental `-Wlifetime` flag),
    [solution](https://godbolt.org/z/bK36rvqh6)
* `std::optional`
  * [problem](https://godbolt.org/z/GoGnb7Tce), [solution](https://godbolt.org/z/oc9ePP853)
  * [problem (memory overhead)](https://godbolt.org/z/G4va5nMYh),
    [solution (niche optimization)](https://godbolt.org/z/h8455oqrh)
  * assignment operator (which delegates to inner type) makes `std::optional<T&>` not feasible
* `std::string`
  * an empty string must still store a terminating null character for
    [`c_str()`](https://en.cppreference.com/w/cpp/string/basic_string/c_str)
* move semantics
  * correctly implementing move constructors and move assignment operators is very hard!
  * [problem (not zero-cost)](https://godbolt.org/z/a6dWWa78P),
    [solution](https://godbolt.org/z/EoMqf6Knx)
* `std::unique_ptr`
  * [problem (not zero-cost)](https://godbolt.org/z/Tfv9rhKj4), [solution](https://godbolt.org/z/jxzYfxfEY)
* `std::shared_ptr`
  * thread-safe? [`std::atomic<std::shared_ptr>`](https://en.cppreference.com/w/cpp/memory/shared_ptr/atomic2)?
* `std::string_view`
  * can be dangling, e.g. when used as local variable or as function output parameter;
    using `std::string_view` as input parameter should be fine
    (and is encouraged if no ownership is desired)
  * methods like `.find()` have to be
    [implemented](https://en.cppreference.com/w/cpp/string/basic_string/find)
    [twice](https://en.cppreference.com/w/cpp/string/basic_string_view/find)
    * [solution (auto-dereferencing)](https://godbolt.org/z/9nfrcv7ns)
* `std::span`
* `enum class`
* `std::variant`

*Simplification Paradox*: New features that are supposed to simplify things
are *added* (as opposed to *replacing* something),
making the language larger and more complex.


## Real Improvements in C++

Just kidding, this is of course never going to happen!

A promising initiative was the "epochs" proposal in 2019/2020,
but it sadly didn't lead anywhere:
[blog post](https://vittorioromeo.info/index/blog/fixing_cpp_with_epochs.html),
[lightning talk](https://youtu.be/PFdKFoQxRqM),
[standard proposal](https://wg21.link/p1881),
[tracking issue](https://github.com/cplusplus/papers/issues/631).


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
* [`std::vector<bool>`](https://en.cppreference.com/w/cpp/container/vector_bool)
