# Rust Overview

**Summary**: Rust is a systems programming language focused on safety, speed, and concurrency — it achieves memory safety without a garbage collector through a compile-time ownership system.

**Sources**: https://doc.rust-lang.org/book/

**Last updated**: 2026-04-16

#rust

---

Rust is designed for systems-level work (operating systems, embedded devices, game engines, networking) where you need control over memory but cannot afford runtime overhead or undefined behavior. Its central innovation is the **ownership system** — a set of compile-time rules that eliminate entire classes of bugs (use-after-free, data races, null pointer dereferences) without any runtime cost.

---

## Core Topics

- [[rust-variables-types]] — Variables, mutability, shadowing, constants, and all scalar and compound data types
- [[rust-functions-control-flow]] — Function syntax, statements vs expressions, return values, `if`, `loop`, `while`, `for`
- [[rust-ownership]] — Ownership rules, move semantics, `Copy` trait, stack vs heap, scope and `drop`
- [[rust-borrowing-references]] — References, borrowing rules, mutable references, dangling reference prevention
- [[rust-structs]] — Defining structs, field init shorthand, update syntax, tuple structs, unit-like structs
- [[rust-enums-pattern-matching]] — Enums with data, `Option<T>`, `match` expressions, `if let`, exhaustive matching
- [[rust-error-handling]] — `Result<T, E>`, `?` operator, `unwrap`, `expect`, error propagation, `panic!`
- [[rust-traits-generics]] — Traits, implementing traits, default implementations, generics, trait bounds, monomorphization
- [[rust-macros]] — What macros are, `println!`, `vec!`, `assert_eq!`, compile-time code generation

---

## Learning Roadmap

See [[rust-learning-roadmap]] for the 2-day sprint plan toward OS development.

**Recommended order:**
1. Variables & Types → Functions & Control Flow (syntax foundation)
2. Ownership → Borrowing & References (the core Rust idea — spend most time here)
3. Structs → Enums & Pattern Matching (how data is shaped)
4. Error Handling (how failures are managed)
5. Traits & Generics (abstraction system)
6. Macros (metaprogramming utilities)

---

## Why Rust is Different

| Feature | Rust | C/C++ | Go/Python |
|---|---|---|---|
| Memory management | Compile-time ownership | Manual `malloc`/`free` | Garbage collector |
| Null safety | `Option<T>` — no null | Null pointers exist | Null/nil exist |
| Error handling | `Result<T, E>` — forced | Exceptions / error codes | Exceptions |
| Concurrency safety | Enforced by compiler | Manual (data races possible) | Runtime checks |
| Runtime overhead | Zero | Zero | GC pauses |

The ownership model is the price of admission — it has a steep learning curve, but it replaces an entire category of runtime bugs with compile-time errors.
