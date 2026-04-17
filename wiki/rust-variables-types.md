# Rust Variables and Types

**Summary**: Rust variables are immutable by default; the type system covers scalar types (integers, floats, bool, char) and compound types (tuples, arrays), with strict rules about overflow, mutability, and shadowing.

**Pre-Req**: Basic programming concepts (variables, types)

**Sources**: https://doc.rust-lang.org/book/ch03-01-variables-and-mutability.html, https://doc.rust-lang.org/book/ch03-02-data-types.html

**Last updated**: 2026-04-16

#rust

---

## Variables and Mutability

By default, all variables in Rust are **immutable** — once a value is bound, it cannot be changed.

```rust
let x = 5;
x = 6;  // ❌ Error: cannot assign twice to immutable variable
```

Add `mut` to allow reassignment:

```rust
let mut x = 5;
x = 6;  // ✅ OK
```

The `mut` keyword is intentional — it signals to both the compiler and future readers that this value will change.

---

## Constants

Constants use `const`, require an explicit type annotation, and must be set to a compile-time expression (not a runtime value). They can be declared in any scope, including global.

```rust
const THREE_HOURS_IN_SECONDS: u32 = 60 * 60 * 3;
```

**Naming convention:** all uppercase with underscores.

| | `let` variable | `const` |
|---|---|---|
| Keyword | `let` | `const` |
| Type annotation | Optional | Required |
| Mutability | Mutable with `mut` | Always immutable |
| Scope | Block scope | Any scope |
| Value | Runtime or compile-time | Compile-time only |

---

## Shadowing

Shadowing lets you reuse a variable name with `let`. The new variable *shadows* the old one — the old value is no longer accessible.

```rust
let x = 5;
let x = x + 1;   // x is now 6

{
    let x = x * 2;  // x is 12 in this inner scope
    println!("{x}");  // 12
}

println!("{x}");  // 6 — back to outer scope
```

**Key advantage over `mut`:** shadowing can change the type of a variable.

```rust
let spaces = "   ";       // &str
let spaces = spaces.len(); // usize — type changed, no error

// With mut this is impossible:
let mut spaces = "   ";
spaces = spaces.len();  // ❌ type mismatch
```

---

## Scalar Types

Scalar types represent a single value.

### Integers

| Length | Signed | Unsigned |
|---|---|---|
| 8-bit | `i8` | `u8` |
| 16-bit | `i16` | `u16` |
| 32-bit | `i32` (default) | `u32` |
| 64-bit | `i64` | `u64` |
| 128-bit | `i128` | `u128` |
| Pointer-sized | `isize` | `usize` |

- Signed range: `-(2^(n-1))` to `2^(n-1) - 1`
- Unsigned range: `0` to `2^n - 1`
- Default integer type is `i32`
- `usize`/`isize` match the pointer width of the target platform (used for indexing)

**Integer literals:**
```rust
let decimal     = 98_222;      // underscores for readability
let hex         = 0xff;
let octal       = 0o77;
let binary      = 0b1111_0000;
let byte        = b'A';        // u8 only
let with_suffix = 57u8;        // type suffix
```

**Integer overflow:**
- **Debug builds:** program panics
- **Release builds:** wraps using two's complement
- Explicit control methods: `wrapping_add`, `checked_add` (returns `Option`), `saturating_add`, `overflowing_add`

### Floating-Point

```rust
let x = 2.0;       // f64 (default — better precision)
let y: f32 = 3.0;  // f32
```

Both are signed, IEEE-754 compliant. `f64` is the default since modern CPUs handle it at similar speed to `f32`.

### Boolean

```rust
let t = true;
let f: bool = false;
```

- 1 byte in size
- Used in `if` and `while` conditions (must be bool — Rust does not coerce integers to bool)

### Character

```rust
let c = 'z';
let heart = '😻';  // Unicode scalar values supported
```

- 4 bytes — represents Unicode scalar values (U+0000 to U+10FFFF, excluding surrogates)
- Single quotes (unlike strings which use double quotes)

---

## Compound Types

Compound types group multiple values into one.

### Tuples

Fixed-length, can hold different types.

```rust
let tup: (i32, f64, u8) = (500, 6.4, 1);

// Destructuring
let (x, y, z) = tup;

// Direct index access
let first = tup.0;
let second = tup.1;
```

The empty tuple `()` is called the **unit type** — it's what functions return when they return nothing.

### Arrays

Fixed-length, all elements must be the same type. Stored on the **stack**.

```rust
let a = [1, 2, 3, 4, 5];
let a: [i32; 5] = [1, 2, 3, 4, 5];  // type annotation: [type; length]
let a = [3; 5];                       // [3, 3, 3, 3, 3] — fill syntax

let first = a[0];
```

Out-of-bounds access **panics at runtime** (Rust checks array bounds — no undefined behavior like C).

Use arrays when the size is fixed and known. Use `Vec` (heap-allocated, growable) when the size may change.

---

## Related Pages

- [[rust-overview]] — full Rust topic map
- [[rust-ownership]] — how types interact with the ownership system (`Copy` vs move)
- [[rust-functions-control-flow]] — using types in function signatures and control flow
