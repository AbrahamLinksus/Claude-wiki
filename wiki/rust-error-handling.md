# Rust Error Handling

**Summary**: Rust separates recoverable errors (`Result<T, E>`) from unrecoverable ones (`panic!`); the `?` operator propagates errors concisely, and Rust forces you to explicitly handle every failure case at compile time.

**Pre-Req**: [[rust-enums-pattern-matching]]

**Sources**: https://doc.rust-lang.org/book/ch09-02-recoverable-errors-with-result.html

**Last updated**: 2026-04-16

#rust

---

## Two Categories of Errors

**Unrecoverable errors — `panic!`**
The program cannot continue. Rust unwinds the stack and prints an error message. Used for bugs, not expected failure cases.

```rust
panic!("something went very wrong");
```

Rust also automatically panics on out-of-bounds array access, integer overflow in debug mode, etc.

**Recoverable errors — `Result<T, E>`**
Expected failures (file not found, bad input, network timeout). The caller gets to decide what to do. This is Rust's primary error handling mechanism.

---

## `Result<T, E>`

`Result` is an enum in the standard library:

```rust
enum Result<T, E> {
    Ok(T),    // success — contains the return value
    Err(E),   // failure — contains the error
}
```

Functions that can fail return `Result`:

```rust
use std::fs::File;

fn main() {
    let result = File::open("hello.txt");
    // result: Result<File, std::io::Error>
}
```

---

## Handling `Result` with `match`

```rust
use std::fs::File;

fn main() {
    let file = match File::open("hello.txt") {
        Ok(f)  => f,
        Err(e) => panic!("Failed to open file: {e:?}"),
    };
}
```

Match on specific error kinds:

```rust
use std::fs::File;
use std::io::ErrorKind;

fn main() {
    let file = match File::open("hello.txt") {
        Ok(f) => f,
        Err(e) => match e.kind() {
            ErrorKind::NotFound => match File::create("hello.txt") {
                Ok(fc) => fc,
                Err(e) => panic!("Could not create file: {e:?}"),
            },
            _ => panic!("Could not open file: {e:?}"),
        },
    };
}
```

---

## `unwrap()` and `expect()`

These are shortcuts for "give me the value or panic":

```rust
// unwrap — panics with a generic message on Err
let file = File::open("hello.txt").unwrap();

// expect — panics with YOUR message on Err (prefer this)
let file = File::open("hello.txt")
    .expect("hello.txt should exist in this project");
```

Use `expect` over `unwrap` in real code — the custom message makes debugging much easier.

**When to use:** Prototypes, tests, and situations where you are certain the operation cannot fail. In production paths, propagate errors instead.

---

## Error Propagation with `?`

The `?` operator propagates errors up to the caller. It is shorthand for:
- If `Ok(value)`, unwrap and continue
- If `Err(e)`, return `Err(e)` immediately from the current function

```rust
use std::fs::File;
use std::io::{self, Read};

// Without ? — verbose
fn read_username_v1() -> Result<String, io::Error> {
    let mut file = match File::open("hello.txt") {
        Ok(f) => f,
        Err(e) => return Err(e),
    };
    let mut s = String::new();
    match file.read_to_string(&mut s) {
        Ok(_) => Ok(s),
        Err(e) => Err(e),
    }
}

// With ? — concise
fn read_username_v2() -> Result<String, io::Error> {
    let mut file = File::open("hello.txt")?;  // return Err if this fails
    let mut s = String::new();
    file.read_to_string(&mut s)?;             // return Err if this fails
    Ok(s)
}

// Even more concise — chained
fn read_username_v3() -> Result<String, io::Error> {
    let mut s = String::new();
    File::open("hello.txt")?.read_to_string(&mut s)?;
    Ok(s)
}

// Or just use the stdlib shortcut
fn read_username_v4() -> Result<String, io::Error> {
    std::fs::read_to_string("hello.txt")
}
```

**`?` also works with `Option<T>`:**

```rust
fn last_char_of_first_line(text: &str) -> Option<char> {
    text.lines().next()?.chars().last()
}
```

### Where `?` Can Be Used

`?` can only be used inside functions that return `Result` or `Option`. Using it in `main()` requires changing its signature:

```rust
use std::error::Error;
use std::fs::File;

fn main() -> Result<(), Box<dyn Error>> {
    let _ = File::open("hello.txt")?;
    Ok(())
}
```

`Box<dyn Error>` means "any error type" — useful for `main` or quick scripts.

### Error Type Conversion

`?` automatically converts error types using the `From` trait. If a function returns `Result<T, MyError>` and an inner call returns `io::Error`, `?` will call `From<io::Error> for MyError` automatically — as long as you've implemented it.

---

## `unwrap_or_else` — Functional Style

An alternative to nested `match` using closures:

```rust
let file = File::open("hello.txt").unwrap_or_else(|e| {
    if e.kind() == ErrorKind::NotFound {
        File::create("hello.txt").unwrap_or_else(|e| {
            panic!("Could not create: {e:?}");
        })
    } else {
        panic!("Could not open: {e:?}");
    }
});
```

---

## Choosing Between `panic!` and `Result`

| Use `panic!` | Use `Result` |
|---|---|
| Bug in your code (should never happen) | Expected failure (file missing, bad input) |
| Prototype / example code | Library functions |
| Tests | Production code paths |
| Invariant violation | Any failure a caller might want to handle |

**Rule of thumb:** If your caller could reasonably want to handle or recover from the failure, use `Result`. If it's a bug or impossible state, use `panic!`.

---

## Related Pages

- [[rust-overview]] — full Rust topic map
- [[rust-enums-pattern-matching]] — `Result` and `Option` are just enums, matched the same way
- [[rust-traits-generics]] — `Box<dyn Error>` and trait objects for dynamic error types
