# C++ Introduction

**Summary**: What C++ is, why it matters for systems programming, and a roadmap for learning it from scratch.

**Sources**: https://en.cppreference.com/w/cpp

**Last updated**: 2026-04-16

---

#program

## What is C++?

C++ is a **compiled, statically-typed, general-purpose programming language** created by Bjarne Stroustrup at Bell Labs in 1979 as an extension of C. It adds:

- **Object-oriented programming** — classes, inheritance, polymorphism
- **Generic programming** — templates
- **Zero-overhead abstractions** — you pay only for what you use
- **Direct memory control** — pointers, manual allocation, no garbage collector

The language is standardized by ISO. Major versions you'll encounter:
- **C++11** — the modern revolution (lambdas, auto, smart pointers, move semantics, threads)
- **C++14** — refinements to C++11
- **C++17** — structured bindings, if constexpr, std::optional, std::variant
- **C++20** — concepts, ranges, coroutines, modules
- **C++23** — latest standard

When you see `(C++11)` or `(C++17)` in these notes, it means the feature was added in that version.

---

## Why C++ for Systems / OS Development?

- **Compiled to machine code** — no runtime interpreter or virtual machine
- **No mandatory garbage collector** — you control when memory is allocated and freed
- **Direct hardware access** — pointer arithmetic, bitwise operations, inline assembly
- **Zero-cost abstractions** — a C++ class costs nothing extra vs raw C structs if you don't use virtual functions
- **C compatibility** — can call C code directly; the Linux kernel is C, and C++ can interface with it

---

## How C++ Programs Work (Big Picture)

```
Source files (.cpp, .h)
        ↓  preprocessor (handles #include, #define)
Preprocessed source
        ↓  compiler (g++, clang++)
Object files (.o)
        ↓  linker (ld)
Executable binary
        ↓  OS loader
Running process
```

Each `.cpp` file compiles independently into an **object file**. The **linker** then combines all object files into a single executable. See [[cpp-program-structure]].

---

## Learning Roadmap

Work through these topics in order:

### Foundation
1. [[cpp-program-structure]] — translation units, the `main` function, compilation
2. [[cpp-fundamental-types]] — int, float, bool, char, void and their sizes
3. [[cpp-declarations-definitions]] — how to introduce names, const, static, extern

### Coming Next
4. Variables, initialization, and assignment
5. Operators and expressions
6. Control flow — if, switch, for, while, do-while
7. Functions — parameters, return values, overloading
8. Pointers and references
9. Arrays and strings
10. Classes and structs
11. Memory management — stack vs heap, new/delete
12. Templates
13. The Standard Library — containers, algorithms, iterators
14. Modern C++ — smart pointers, move semantics, lambdas

---

## Compiling Your First Program

Install `g++` (GCC's C++ compiler) or `clang++`:

```bash
# Compile
g++ -std=c++17 -Wall -o hello hello.cpp

# Run
./hello
```

Flags explained:
- `-std=c++17` — use the C++17 standard
- `-Wall` — enable all common warnings (always use this)
- `-o hello` — name the output binary `hello`

A minimal program:

```cpp
#include <iostream>   // brings in std::cout

int main() {
    std::cout << "Hello, World!\n";
    return 0;         // 0 = success
}
```

---

## Related Pages

- [[cpp-program-structure]]
- [[cpp-fundamental-types]]
- [[cpp-declarations-definitions]]
