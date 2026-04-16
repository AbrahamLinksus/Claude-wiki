# C++ Fundamental Types

**Summary**: All built-in types in C++ — integers, floats, bool, char, void — their sizes, value ranges, and when to use each.

**Pre-Req**: [[cpp-program-structure]]

**Sources**: https://en.cppreference.com/w/cpp/language/types

**Last updated**: 2026-04-16

---

#program

## Overview

Every variable in C++ has a **type** — it tells the compiler:
- How many bytes of memory to allocate
- How to interpret those bytes (signed integer? floating point? text character?)
- What operations are valid (you can multiply two `int`s but not two `bool`s)

C++ has two layers of types:
1. **Fundamental (built-in) types** — provided by the language itself. Covered on this page.
2. **Compound types** — built from fundamental types (arrays, pointers, references, structs, classes). Covered later.

---

## Integer Types

Integers hold **whole numbers** — no decimal point.

### The Core Integer Types

```cpp
short int       // or just: short
int
long int        // or just: long
long long int   // or just: long long  (C++11)
```

The C++ standard only guarantees **minimum sizes**, not exact sizes:

| Type | Minimum bits | Typical (modern 64-bit) |
|---|---|---|
| `short` | 16 | 16 bits (2 bytes) |
| `int` | 16 | 32 bits (4 bytes) |
| `long` | 32 | 64 bits on Linux/macOS, 32 bits on Windows |
| `long long` | 64 | 64 bits (8 bytes) |

The standard guarantees: `sizeof(short) ≤ sizeof(int) ≤ sizeof(long) ≤ sizeof(long long)`.

### Signed vs Unsigned

Every integer type comes in two flavors:

```cpp
int x = -5;          // signed — can hold negative values (default)
unsigned int y = 5;  // unsigned — only non-negative values, larger positive range
```

For an N-bit type:
- **Signed**: range is −2^(N-1) to +2^(N-1) − 1
- **Unsigned**: range is 0 to 2^N − 1

Value ranges for common sizes:

| Type | Bits | Min | Max |
|---|---|---|---|
| `signed char` | 8 | −128 | 127 |
| `unsigned char` | 8 | 0 | 255 |
| `short` | 16 | −32,768 | 32,767 |
| `unsigned short` | 16 | 0 | 65,535 |
| `int` | 32 | −2,147,483,648 | 2,147,483,647 |
| `unsigned int` | 32 | 0 | 4,294,967,295 |
| `long long` | 64 | −9.2 × 10^18 | 9.2 × 10^18 |
| `unsigned long long` | 64 | 0 | 1.8 × 10^19 |

### Integer Overflow

Signed integer overflow is **undefined behavior** in C++ — the compiler is allowed to assume it never happens, which can cause subtle bugs:

```cpp
int x = 2147483647;
x++;                  // undefined behavior! Do not rely on this wrapping to -2147483648
```

Unsigned overflow is **well-defined** — it wraps around modulo 2^N:

```cpp
unsigned int x = 4294967295u;
x++;  // wraps to 0 — this is guaranteed
```

### Fixed-Width Integer Types (`<cstdint>`)

When you need an **exact size** (common in OS dev, networking, file formats), use these:

```cpp
#include <cstdint>

int8_t    uint8_t    // exactly 8 bits
int16_t   uint16_t   // exactly 16 bits
int32_t   uint32_t   // exactly 32 bits
int64_t   uint64_t   // exactly 64 bits
```

Also useful:
```cpp
size_t      // unsigned type for sizes and indices — result of sizeof()
ptrdiff_t   // signed type for pointer differences
intptr_t    // signed integer the same size as a pointer
uintptr_t   // unsigned integer the same size as a pointer
```

**Rule of thumb**: use `int` for general counting/indexing; use fixed-width types when the exact size matters (hardware registers, network packets, file formats, OS dev).

---

## Floating-Point Types

Floating-point types hold **decimal numbers** using IEEE-754 standard representation.

```cpp
float        // 32 bits — single precision (~7 significant digits)
double       // 64 bits — double precision (~15 significant digits)
long double  // 80 bits on x86 Linux, platform-dependent elsewhere
```

### Value Ranges

| Type | Bits | Precision | Approximate range |
|---|---|---|---|
| `float` | 32 | ~7 digits | ±1.2×10^-38 to ±3.4×10^38 |
| `double` | 64 | ~15 digits | ±2.2×10^-308 to ±1.8×10^308 |
| `long double` | 80 (x86) | ~18-19 digits | ±3.4×10^-4932 to ±1.2×10^4932 |

### Floating-Point Literals

```cpp
float  f = 3.14f;     // 'f' suffix required for float
double d = 3.14;      // no suffix = double (default)
double e = 1.5e10;    // scientific notation: 1.5 × 10^10
```

### Special Values (IEEE-754)

```cpp
#include <cmath>
double a = INFINITY;      // positive infinity
double b = -INFINITY;     // negative infinity
double c = NAN;           // not a number (result of 0.0/0.0 etc.)
double d = -0.0;          // negative zero — equal to +0.0 but different bit pattern
```

### Floating-Point Pitfall — Never Compare with `==`

```cpp
double x = 0.1 + 0.2;
if (x == 0.3)             // FALSE — floating point is not exact!
    std::cout << "equal";

// Correct way:
#include <cmath>
if (std::abs(x - 0.3) < 1e-9)
    std::cout << "close enough";
```

This happens because 0.1 and 0.2 cannot be represented exactly in binary — the actual stored values have tiny rounding errors.

### When to Use Which

- `float` — graphics, large arrays where memory matters, speed-critical math that doesn't need precision
- `double` — default for most calculations
- `long double` — very high precision scientific computing (rare)

---

## Boolean Type

```cpp
bool flag = true;
bool empty = false;
```

- Stores `true` (1) or `false` (0)
- `sizeof(bool)` is implementation-defined but usually 1 byte
- Any non-zero integer converts to `true`; zero converts to `false`
- `true` converts to `1`, `false` to `0` when used in arithmetic

```cpp
bool a = 5;       // a = true (non-zero)
bool b = 0;       // b = false
int x = true;     // x = 1
int y = false;    // y = 0
```

---

## Character Types

Character types store **text characters** as small integers mapped to a character encoding (usually ASCII or Unicode).

```cpp
char       // 1 byte — platform-dependent signedness. Used for ASCII text.
signed char    // 1 byte — explicitly signed (-128 to 127)
unsigned char  // 1 byte — explicitly unsigned (0 to 255). Used for raw bytes.
```

Unicode character types:
```cpp
wchar_t    // wide character — 16 bits on Windows, 32 bits on Linux
char8_t    // UTF-8 code unit, 8 bits (C++20)
char16_t   // UTF-16 code unit, 16 bits (C++11)
char32_t   // UTF-32 code unit, 32 bits (C++11)
```

Character literals:
```cpp
char a = 'A';          // ASCII 65
char newline = '\n';   // escape sequence
char tab = '\t';
char null = '\0';      // null terminator — ends C-style strings
unsigned char byte = 0xFF;   // raw byte value 255
```

Under the hood, `char` is just a small integer:
```cpp
char c = 'A';
int i = c;     // i = 65 — the ASCII code for 'A'
c++;           // c = 'B' — arithmetic on chars is valid
```

**Important**: for raw bytes and binary data, always use `unsigned char` or `uint8_t`. Using plain `char` for binary data can cause sign-extension bugs.

---

## void

`void` means "no type" or "no value". You cannot create a variable of type `void`, but you use it in two ways:

### 1. Function return type — the function returns nothing

```cpp
void printHello() {
    std::cout << "Hello\n";
    // no return statement needed (or just: return;)
}
```

### 2. `void*` — generic pointer

A `void*` pointer can point to any type. You must cast before using it.

```cpp
void* ptr = nullptr;
int x = 42;
ptr = &x;                   // store pointer to int in void*
int* p = (int*)ptr;         // cast back before using
std::cout << *p;            // 42
```

`void*` is common in C but mostly replaced by templates in modern C++.

---

## `nullptr` and `std::nullptr_t` (C++11)

The null pointer constant:

```cpp
int* p = nullptr;           // pointer to nothing
```

Before C++11, `NULL` (a macro for `0`) was used. `nullptr` is safer because it has its own type (`std::nullptr_t`) and won't accidentally convert to `int`.

---

## Checking Type Sizes at Runtime

```cpp
#include <iostream>

int main() {
    std::cout << "bool:        " << sizeof(bool)        << " bytes\n";
    std::cout << "char:        " << sizeof(char)        << " bytes\n";
    std::cout << "short:       " << sizeof(short)       << " bytes\n";
    std::cout << "int:         " << sizeof(int)         << " bytes\n";
    std::cout << "long:        " << sizeof(long)        << " bytes\n";
    std::cout << "long long:   " << sizeof(long long)   << " bytes\n";
    std::cout << "float:       " << sizeof(float)       << " bytes\n";
    std::cout << "double:      " << sizeof(double)      << " bytes\n";
    std::cout << "pointer:     " << sizeof(void*)       << " bytes\n";
}
```

---

## Type Aliases

You can create alternative names for types:

```cpp
using Byte = unsigned char;    // C++11 style (preferred)
typedef unsigned char Byte;    // old C style (still valid)

Byte buffer[1024];             // same as: unsigned char buffer[1024]
```

---

## Quick Selection Guide

| Situation | Type to use |
|---|---|
| General integer counting/indexing | `int` |
| Loop index over a container size | `size_t` |
| Exact bit width needed (hardware, files, network) | `uint8_t`, `uint32_t`, etc. |
| True/false flag | `bool` |
| ASCII text characters | `char` |
| Raw bytes / binary data | `unsigned char` or `uint8_t` |
| General decimal math | `double` |
| Large arrays of floats (graphics) | `float` |
| Pointer arithmetic | `ptrdiff_t` |
| Storing a pointer as an integer | `uintptr_t` |

---

## Related Pages

- [[cpp-introduction]]
- [[cpp-program-structure]]
- [[cpp-declarations-definitions]]
