# C++ Program Structure

**Summary**: How a C++ program is laid out — source files, translation units, the main function, scope, linkage, and how the compiler + linker turn it into a binary.

**Pre-Req**: [[cpp-introduction]]

**Sources**: https://en.cppreference.com/w/cpp/language/basic_concepts

**Last updated**: 2026-04-16

---

#program

## A C++ Program is a Collection of Text Files

A C++ program is built from one or more **source files** (`.cpp`) and **header files** (`.h` or `.hpp`). You write these, and a compiler translates them into machine code.

```
myproject/
├── main.cpp        ← contains int main()
├── math.cpp        ← some functions
├── math.h          ← declarations for math.cpp
└── utils.cpp       ← more functions
```

---

## Translation Units

When you compile `main.cpp`, the **preprocessor** first runs and:
- Replaces every `#include "math.h"` with the full contents of `math.h`
- Expands `#define` macros
- Strips comments

The result is called a **translation unit** — the single merged file that the compiler actually sees. Each `.cpp` produces one translation unit, which compiles into one **object file** (`.o`).

```
main.cpp  → (preprocessor) → translation unit → (compiler) → main.o
math.cpp  → (preprocessor) → translation unit → (compiler) → math.o
utils.cpp → (preprocessor) → translation unit → (compiler) → utils.o
                                                               ↓
                                                          (linker)
                                                               ↓
                                                          executable
```

The **linker** stitches all the `.o` files together, resolves cross-file references, and produces the final binary.

---

## The `main` Function

Every C++ program must have exactly **one** `main` function. It is the entry point — execution starts here.

```cpp
int main() {
    // your code
    return 0;
}
```

Or with command-line arguments:

```cpp
int main(int argc, char* argv[]) {
    // argc = number of arguments
    // argv = array of argument strings
    return 0;
}
```

The return value is the **exit code** sent to the operating system:
- `0` = success
- Any non-zero = error

You can also use `return EXIT_SUCCESS` and `return EXIT_FAILURE` from `<cstdlib>`.

---

## Entities

A C++ program is built from **entities** — things that have names and types:

| Entity | Example |
|---|---|
| Variable | `int x = 5;` |
| Function | `int add(int a, int b);` |
| Type | `struct Point { int x, y; };` |
| Namespace | `namespace std { }` |
| Template | `template<typename T> class Vec;` |

Preprocessor macros (`#define`) are **not** entities — they are textual substitutions that happen before the compiler even runs.

---

## Scope — Where Names Are Visible

**Scope** is the region of code where a name can be used. Once you leave that region, the name no longer exists.

### Block Scope
Inside `{ }` — the most common. Variables declared here die when the block ends.

```cpp
{
    int x = 10;     // x lives here
    std::cout << x; // OK
}
// x is gone here — using it would be a compile error
```

### Function Scope
Labels (for `goto`) are function-scoped — they're visible throughout the function but nowhere else.

### Namespace Scope
Names declared at the top level of a file or inside a `namespace` block. These live for the duration of the program.

```cpp
int globalVar = 100;   // namespace scope (global)

namespace MyLib {
    int helper = 5;    // namespace scope (inside MyLib)
}
```

### Class Scope
Names declared inside a `class` or `struct`. Accessed through the object or `::` operator.

```cpp
class Dog {
    int age;     // class scope — only accessible via a Dog object
};
```

---

## Name Lookup

When the compiler sees a name, it performs **name lookup** — finding which declaration that name refers to.

### Unqualified lookup
Just a bare name — compiler searches outward from the current scope.

```cpp
int x = 5;

void foo() {
    std::cout << x;   // unqualified lookup: finds global x
}
```

### Qualified lookup
Using `::` or `.` or `->` to explicitly specify where to look.

```cpp
std::cout          // look in namespace std for cout
myObject.value     // look in myObject for value
MyClass::method()  // look in MyClass for method
```

---

## Storage Duration — How Long Does an Object Live?

| Duration | Where declared | Lifetime |
|---|---|---|
| **Automatic** | Inside a function/block | Born at declaration, dies at end of block |
| **Static** | Global, or `static` local | Born at program start (or first use), dies at program end |
| **Dynamic** | `new` expression | Born at `new`, dies at `delete` |
| **Thread-local** | `thread_local` keyword | One copy per thread |

```cpp
int global = 10;          // static duration

void foo() {
    int local = 5;        // automatic duration
    static int count = 0; // static duration (but local scope)
    count++;              // persists between calls to foo()

    int* p = new int(42); // dynamic duration
    delete p;             // must manually free
}
```

---

## Linkage — Visible Across Files?

**Linkage** controls whether a name in one translation unit can refer to the same entity defined in another.

### External linkage
The name is visible to the linker — other `.cpp` files can reference it.

```cpp
// file1.cpp
int counter = 0;          // external linkage by default

// file2.cpp
extern int counter;       // tells the compiler: this is defined elsewhere
counter++;                // links to file1.cpp's counter
```

### Internal linkage
Only visible within the current `.cpp` file. Use `static` or an anonymous namespace.

```cpp
static int helper = 5;    // internal linkage — file2.cpp cannot see this

namespace {
    int secret = 99;      // also internal linkage (preferred modern way)
}
```

### No linkage
Local variables — only visible in their block, invisible to the linker entirely.

---

## The One Definition Rule (ODR)

A core rule in C++: **every entity must be defined exactly once across the entire program.**

- You can **declare** something many times (in every `.cpp` that needs it)
- You can only **define** it once

```cpp
// math.h
extern int result;          // declaration — put this in header files
int add(int a, int b);      // declaration

// math.cpp
int result = 0;             // definition — exactly once
int add(int a, int b) {     // definition — exactly once
    return a + b;
}

// main.cpp
#include "math.h"
// uses result and add() — no definitions here, just the declarations from math.h
```

Violating ODR (defining something twice) is a **linker error**.

---

## Header Guards

Because headers get `#include`d into multiple translation units, you need to prevent their contents from being pasted twice. Use a **header guard**:

```cpp
// math.h
#ifndef MATH_H
#define MATH_H

int add(int a, int b);

#endif
```

Or the modern, simpler form:

```cpp
// math.h
#pragma once    // supported by all major compilers

int add(int a, int b);
```

---

## Related Pages

- [[cpp-introduction]]
- [[cpp-fundamental-types]]
- [[cpp-declarations-definitions]]
