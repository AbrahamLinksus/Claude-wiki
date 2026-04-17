# Rust Functions and Control Flow

**Summary**: Rust functions use `fn`, require explicit parameter types, and return the final expression implicitly; control flow covers `if` as an expression, and three loop types: `loop`, `while`, and `for`.

**Pre-Req**: [[rust-variables-types]]

**Sources**: https://doc.rust-lang.org/book/ch03-03-how-functions-work.html, https://doc.rust-lang.org/book/ch03-05-control-flow.html

**Last updated**: 2026-04-16

#rust

---

## Functions

Functions are declared with `fn`. Rust uses **snake_case** for function and variable names. Definition order does not matter — Rust finds functions declared anywhere in the file.

```rust
fn main() {
    another_function();
}

fn another_function() {
    println!("Another function.");
}
```

### Parameters

All parameters **must** have explicit type annotations. Multiple parameters are comma-separated.

```rust
fn print_measurement(value: i32, unit: char) {
    println!("The measurement is: {value}{unit}");
}
```

### Statements vs Expressions

This is critical in Rust:

- **Statements** perform an action and **do not return a value** (end with `;`)
- **Expressions** evaluate to a value (no trailing `;`)

```rust
let y = {
    let x = 3;
    x + 1       // expression — no semicolon — this evaluates to 4
};              // y = 4
```

Adding a `;` to `x + 1` makes it a statement, returning `()` (the unit type), not `4`.

### Return Values

Declare the return type with `->`. The function's return value is the **last expression** (no semicolon).

```rust
fn five() -> i32 {
    5   // implicit return — no semicolon
}

fn plus_one(x: i32) -> i32 {
    x + 1
}
```

For early returns, use the `return` keyword:

```rust
fn check(x: i32) -> i32 {
    if x < 0 {
        return 0;   // early return
    }
    x * 2
}
```

**Common mistake:** adding `;` to the last line turns the expression into a statement, causing a type mismatch:

```rust
fn plus_one(x: i32) -> i32 {
    x + 1;  // ❌ returns () instead of i32
}
```

---

## Control Flow

### `if` Expressions

The condition must be a `bool` — Rust does not coerce other types.

```rust
if number < 5 {
    println!("small");
} else if number == 5 {
    println!("five");
} else {
    println!("large");
}
```

`if` is an **expression** in Rust, so it can be used on the right-hand side of `let`:

```rust
let number = if condition { 5 } else { 6 };
```

Both arms must return the **same type** — Rust needs to know the type of `number` at compile time.

```rust
let x = if condition { 5 } else { "six" };  // ❌ mismatched types
```

---

### `loop` — Infinite Loop

Runs forever until a `break`. Can return a value via `break`:

```rust
let mut counter = 0;

let result = loop {
    counter += 1;
    if counter == 10 {
        break counter * 2;  // returns 20
    }
};
// result == 20
```

**Loop labels** allow breaking out of nested loops:

```rust
'outer: loop {
    loop {
        break 'outer;  // exits the outer loop
    }
}
```

Labels begin with a single quote: `'label_name`.

---

### `while` — Conditional Loop

Runs while a condition is true:

```rust
let mut n = 3;
while n != 0 {
    println!("{n}!");
    n -= 1;
}
```

---

### `for` — Collection Iteration

The most idiomatic and safe loop in Rust. No manual indexing, no bounds errors.

```rust
let a = [10, 20, 30, 40, 50];

for element in a {
    println!("{element}");
}
```

Using a **range**:

```rust
for n in 1..4 {          // 1, 2, 3 (exclusive end)
    println!("{n}");
}

for n in (1..4).rev() {  // 3, 2, 1 — reversed
    println!("{n}!");
}
```

**Prefer `for` over `while` for collections** — `while` with manual indexing is error-prone and slower (bounds check every iteration).

---

## Summary

| Construct | Use when |
|---|---|
| `fn name() -> T` | Defining a function returning T |
| Last expression without `;` | Implicit return value |
| `return value;` | Early return |
| `if cond { } else { }` | Conditional branching |
| `let x = if ... { } else { }` | Assigning based on condition |
| `loop { break val; }` | Loop until explicit break, optionally return value |
| `while condition { }` | Repeat while condition holds |
| `for item in collection { }` | Iterate over a collection or range |

---

## Related Pages

- [[rust-overview]] — full Rust topic map
- [[rust-variables-types]] — types used in function signatures
- [[rust-enums-pattern-matching]] — `match` as the more powerful alternative to `if/else` chains
