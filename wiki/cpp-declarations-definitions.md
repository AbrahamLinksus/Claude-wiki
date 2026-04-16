# C++ Declarations and Definitions

**Summary**: The difference between declaring and defining something, the One Definition Rule, type qualifiers (const, volatile), and storage class specifiers (static, extern, auto, thread_local).

**Pre-Req**: [[cpp-program-structure]], [[cpp-fundamental-types]]

**Sources**: https://en.cppreference.com/w/cpp/language/declarations

**Last updated**: 2026-04-16

---

#program

## Declaration vs Definition — The Most Important Distinction

This is one of the most confusing things for beginners. Get this right and a lot of C++ makes sense.

A **declaration** says: *"this name exists, and here is its type."*
A **definition** says: *"this is the actual thing — allocate the memory / generate the code."*

Every definition is also a declaration. Not every declaration is a definition.

```cpp
// DECLARATIONS — just introducing the name
extern int x;               // x exists somewhere, it's an int
void foo(int a, int b);     // foo exists somewhere, takes two ints, returns void

// DEFINITIONS — the actual thing
int x = 10;                 // allocate memory for x, store 10 in it
void foo(int a, int b) {    // generate the code for foo
    std::cout << a + b;
}
```

### Why Does This Distinction Exist?

Because C++ compiles one file at a time. When `main.cpp` calls a function defined in `math.cpp`, the compiler compiling `main.cpp` doesn't have `math.cpp` open. It just needs to know *the function exists and what its signature is* — that's what the declaration (usually in a header file) provides. The linker later connects the call to the actual definition.

---

## The One Definition Rule (ODR)

**Rule**: Every entity must be defined **exactly once** across the entire program.

- Declarations: can repeat as many times as needed
- Definitions: exactly once per program (for most things)

```cpp
// header.h — declarations only
extern int score;
int calculate(int a, int b);

// game.cpp — definitions
int score = 0;                    // defined here — once
int calculate(int a, int b) {     // defined here — once
    return a + b;
}

// main.cpp
#include "header.h"               // gets the declarations
// score and calculate() are declared, defined elsewhere
```

**Violating ODR** = linker error: "multiple definition of X".

**Exception**: `inline` functions and class definitions may appear in multiple translation units as long as they are identical in all of them (this is how header-only libraries work).

---

## Variable Declarations in Detail

The syntax of a declaration is:

```
[storage-class] [qualifiers] type name [= initial-value];
```

Examples:

```cpp
int x;                    // declaration + definition (no initializer — value undefined!)
int x = 5;                // declaration + definition + initialization
int a, b, c;              // three variables in one statement
int* p = nullptr;         // pointer to int
const int MAX = 100;      // constant int
static int count = 0;     // static storage
extern int global;        // declaration only — defined in another file
```

**Always initialize your variables.** In C++, uninitialized local variables contain garbage — whatever bytes happened to be at that memory address. Reading them is undefined behavior.

```cpp
int x;           // x has an indeterminate value — dangerous!
int y = 0;       // safe — explicitly initialized
int z{};         // also safe — value-initialized to 0 (C++11 brace syntax)
```

---

## Type Qualifiers

Qualifiers modify how a variable can be used without changing its underlying type.

### `const` — Cannot Change After Initialization

```cpp
const int MAX = 100;
MAX = 200;           // COMPILE ERROR — const cannot be modified

const double PI = 3.14159;
```

`const` is enforced by the compiler — there is no runtime overhead.

**`const` with pointers** (read right-to-left from the variable name):

```cpp
int value = 42;

const int* p1 = &value;    // pointer to const int
                            // you can change where p1 points
                            // you CANNOT change *p1 (the value)
*p1 = 99;                  // COMPILE ERROR
p1 = nullptr;              // OK

int* const p2 = &value;    // const pointer to int
                            // you CANNOT change where p2 points
                            // you CAN change *p2 (the value)
*p2 = 99;                  // OK
p2 = nullptr;              // COMPILE ERROR

const int* const p3 = &value; // const pointer to const int — nothing can change
```

**Tip**: read the type right-to-left to decode pointer const-ness:
- `const int*` → pointer to (const int) → the int is const
- `int* const` → const pointer to (int) → the pointer is const
- `const int* const` → const pointer to (const int) → both are const

### `volatile` — Don't Optimize This

Tells the compiler: *"this variable may change at any time outside normal program flow — never cache it in a register, always read/write memory directly."*

Used for:
- Memory-mapped hardware registers (the value changes when hardware writes to it)
- Variables shared between signal handlers and main code

```cpp
volatile int* hardware_register = (volatile int*)0x40021000;
*hardware_register = 1;   // compiler will NOT optimize this away
int val = *hardware_register;  // compiler will always read from memory
```

**In normal application code you almost never use `volatile`.**

### `const volatile` Together

Both qualifiers together — the variable is read-only from your code's perspective but may still change from external sources (read-only hardware status registers):

```cpp
const volatile int* status_reg = (const volatile int*)0x40021004;
int s = *status_reg;      // reads every time (volatile)
*status_reg = 1;          // COMPILE ERROR (const)
```

---

## Storage Class Specifiers

These control **where** an object lives in memory and **how long** it lives.

### `auto` — Automatic Storage (C++11: Type Deduction)

Before C++11, `auto` was a keyword meaning automatic storage duration (the default for local variables — so it was useless). In C++11 and later, it means **deduce the type from the initializer**:

```cpp
auto x = 42;             // x is int
auto y = 3.14;           // y is double
auto z = "hello";        // z is const char*
auto w = std::string("world");  // w is std::string
```

`auto` doesn't mean the variable has no type — the compiler always knows the exact type. It just saves you from writing it out.

```cpp
std::vector<int>::iterator it = v.begin();   // verbose
auto it = v.begin();                          // same thing, much cleaner
```

**Use `auto` freely** for complex types; avoid it when the type is short and obvious (`int`, `bool`) because it makes code harder to read at a glance.

### `static` — Two Different Meanings

**Inside a function**: the variable lives for the entire program duration and retains its value between calls.

```cpp
void count_calls() {
    static int count = 0;   // initialized once, then persists
    count++;
    std::cout << "Called " << count << " times\n";
}

count_calls();  // "Called 1 times"
count_calls();  // "Called 2 times"
count_calls();  // "Called 3 times"
```

**At file scope (outside all functions)**: gives the name **internal linkage** — it is invisible to other translation units.

```cpp
// file1.cpp
static int helper = 5;     // only file1.cpp can see this
                            // file2.cpp has no access even with extern
```

Prefer anonymous namespaces over `static` for internal linkage in modern C++:

```cpp
namespace {
    int helper = 5;        // same effect — internal linkage
}
```

### `extern` — Defined Somewhere Else

Declares a variable or function without defining it. Tells the linker: *"look for this in another translation unit."*

```cpp
// config.cpp
int screen_width = 1920;    // definition

// renderer.cpp
extern int screen_width;    // declaration — links to config.cpp
std::cout << screen_width;  // works!
```

Always put `extern` declarations in header files so every file that needs them gets the same declaration.

### `thread_local` (C++11) — One Copy Per Thread

Each thread gets its own independent copy of the variable.

```cpp
thread_local int thread_id = 0;

void worker(int id) {
    thread_id = id;           // only this thread's copy changes
    std::cout << thread_id;
}
```

### `mutable` — Writable Inside a `const` Method

Allows a class member to be modified even when the containing object is `const`. Used for caches and lazy computation.

```cpp
class ExpensiveCalc {
    mutable int cached_result = -1;   // can be changed even in const methods
    mutable bool has_cache = false;

public:
    int result() const {
        if (!has_cache) {
            cached_result = do_heavy_computation();  // modifying mutable member
            has_cache = true;
        }
        return cached_result;
    }
};
```

---

## `decltype` — Deduce a Type From an Expression (C++11)

Like `auto` but deduces the type of an expression without evaluating it:

```cpp
int x = 5;
decltype(x) y = 10;      // y is int (same type as x)

int foo();
decltype(foo()) z;       // z is int (return type of foo)
```

Mostly used in templates and generic code.

---

## Summary Table

| Specifier | Effect |
|---|---|
| `const` | Variable cannot be modified after initialization |
| `volatile` | Never cache in register; always read/write memory |
| `auto` (C++11) | Deduce type from initializer |
| `static` (in function) | Variable persists between calls |
| `static` (at file scope) | Internal linkage — invisible to other files |
| `extern` | Declare without defining; links to another translation unit |
| `thread_local` | One copy per thread |
| `mutable` | Writable inside const member functions |

---

## Related Pages

- [[cpp-introduction]]
- [[cpp-program-structure]]
- [[cpp-fundamental-types]]
