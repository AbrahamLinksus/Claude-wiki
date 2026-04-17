# Rust Macros

**Summary**: Macros are a form of metaprogramming in Rust — they write code that generates other code at compile time, before the compiler processes your program.

**Pre-Req**: Basic Rust syntax, functions, types

**Sources**: `oWiki/Rust from start.md`

**Last updated**: 2026-04-16

---

## What is a Macro?

A macro is code that generates code at **compile time**. When you call a macro, the Rust compiler expands it into real code before anything is compiled or run.

You can always identify a macro by the `!` at the end of its name:

```rust
println!("hello");   // macro — note the !
```

This `!` convention exists specifically to distinguish macros from regular function calls, because they behave differently.

---

## Why Not Just Use Functions?

Rust functions have strict rules — every parameter must have a fixed type and count. Macros bypass this restriction.

`println!` is the clearest example: it can accept zero, one, or many arguments of completely different types:

```rust
println!();                          // zero args — just a newline
println!("hello");                   // one arg — a string literal
println!("x = {}", 42);             // two args — format string + value
println!("{} + {} = {}", 1, 2, 3);  // four args
```

A regular function `fn println(...)` could not handle all these signatures. Macros can, because they operate before the type system kicks in.

---

## How `println!` Works

`println!` takes a **format string** as its first argument, with `{}` as placeholders, then the values to substitute in order:

```rust
let name = "Jake";
let age = 20;
println!("My name is {} and I am {} years old.", name, age);
// → My name is Jake and I am 20 years old.
```

The `{}` placeholder calls the `Display` trait on the value — it's asking Rust to print it in a human-readable way.

There is also `{:?}` for **debug printing**, which works on more types:

```rust
let v = vec![1, 2, 3];
println!("{:?}", v);   // → [1, 2, 3]
```

---

## Common Macros You Will See

| Macro | What it does |
|---|---|
| `println!(...)` | Print to stdout with a newline |
| `print!(...)` | Print to stdout without a newline |
| `eprintln!(...)` | Print to stderr |
| `format!(...)` | Like `println!` but returns a `String` instead of printing |
| `vec![1, 2, 3]` | Creates a `Vec` with initial values |
| `panic!("msg")` | Crashes the program with a message |
| `assert_eq!(a, b)` | Checks two values are equal — used in tests |
| `todo!()` | Marks unimplemented code, panics if reached |

---

## Key Mental Model

> Macros are **compile-time code generation**, not runtime function calls.

When your program runs, the macro is already gone — it was replaced by the code it generated. The `!` is your signal that the compiler is doing extra work before your program even starts.

This is different from most languages where macros are simple text substitution (like C's `#define`). Rust macros are **hygienic** — they understand scope and types, so they can't accidentally break surrounding code.

---

## Related Pages

- [[rust-learning-roadmap]] — where macros fit in the overall Rust learning path
