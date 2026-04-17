# Rust Traits and Generics

**Summary**: Traits define shared behavior (like interfaces); generics allow code to work over many types; together they enable zero-cost abstraction — compile-time polymorphism with no runtime overhead.

**Pre-Req**: [[rust-structs]], [[rust-enums-pattern-matching]]

**Sources**: https://doc.rust-lang.org/book/ch10-01-syntax.html, https://doc.rust-lang.org/book/ch10-02-traits.html

**Last updated**: 2026-04-16

#rust

---

## Generics

Generics let you write code once and use it with many different types, eliminating duplication.

### Generic Functions

Type parameters go in angle brackets `<T>` after the function name:

```rust
fn largest<T: PartialOrd>(list: &[T]) -> &T {
    let mut largest = &list[0];
    for item in list {
        if item > largest {
            largest = item;
        }
    }
    largest
}

fn main() {
    let numbers = vec![34, 50, 25, 100, 65];
    println!("{}", largest(&numbers));   // works with i32

    let chars = vec!['y', 'm', 'a', 'q'];
    println!("{}", largest(&chars));     // works with char
}
```

The `PartialOrd` bound is required because `>` doesn't work on all types — only on types that support ordering.

### Generic Structs

```rust
// Single type parameter — both fields must be same type
struct Point<T> {
    x: T,
    y: T,
}

let int_point = Point { x: 5, y: 10 };
let float_point = Point { x: 1.0, y: 4.0 };

// Multiple type parameters — fields can differ
struct Point<T, U> {
    x: T,
    y: U,
}

let mixed = Point { x: 5, y: 4.0 };  // i32, f64
```

### Generic Enums

You've already used these — `Option<T>` and `Result<T, E>` are generic enums:

```rust
enum Option<T> {
    Some(T),
    None,
}

enum Result<T, E> {
    Ok(T),
    Err(E),
}
```

### Generic Methods

Declare the type parameter on `impl` and use it in the method signatures:

```rust
impl<T> Point<T> {
    fn x(&self) -> &T {
        &self.x
    }
}
```

Implement methods only for a specific concrete type:

```rust
impl Point<f32> {
    fn distance_from_origin(&self) -> f32 {
        (self.x.powi(2) + self.y.powi(2)).sqrt()
    }
    // This method only exists on Point<f32>, not Point<i32>
}
```

---

## Zero-Cost Generics: Monomorphization

Rust generics have **zero runtime cost**. The compiler replaces each generic with its concrete types at compile time — a process called **monomorphization**.

```rust
let x = Some(5);    // becomes Option_i32::Some(5)
let y = Some(5.0);  // becomes Option_f64::Some(5.0)
```

The compiled binary contains separate, specialized versions for each type used — exactly as if you'd written the code by hand. No boxing, no vtable, no dynamic dispatch for generics.

---

## Traits

A **trait** defines a set of method signatures that a type must implement. Think of it as a contract: "any type implementing this trait can do these things."

### Defining a Trait

```rust
pub trait Summary {
    fn summarize(&self) -> String;
}
```

### Implementing a Trait on a Type

```rust
pub struct Article {
    pub headline: String,
    pub author: String,
    pub content: String,
}

impl Summary for Article {
    fn summarize(&self) -> String {
        format!("{}, by {}", self.headline, self.author)
    }
}

pub struct Post {
    pub username: String,
    pub content: String,
}

impl Summary for Post {
    fn summarize(&self) -> String {
        format!("{}: {}", self.username, self.content)
    }
}
```

Use it:

```rust
let post = Post { username: String::from("jake"), content: String::from("hello") };
println!("{}", post.summarize());
```

### The Orphan Rule

You can only implement a trait on a type if **at least one of the trait or the type is local to your crate**. You cannot implement an external trait on an external type. This prevents conflicting implementations across crates.

---

## Default Implementations

Traits can provide default method bodies that implementing types can use as-is or override:

```rust
pub trait Summary {
    fn summarize_author(&self) -> String;

    fn summarize(&self) -> String {
        format!("(Read more from {}...)", self.summarize_author())
    }
}

impl Summary for Post {
    fn summarize_author(&self) -> String {
        format!("@{}", self.username)
    }
    // summarize() uses the default — no need to implement it
}
```

---

## Traits as Parameters

### `impl Trait` Syntax

```rust
pub fn notify(item: &impl Summary) {
    println!("Breaking news! {}", item.summarize());
}
```

Accepts any type implementing `Summary`.

### Trait Bound Syntax

More explicit form — required when you need to constrain multiple parameters to the **same** concrete type:

```rust
pub fn notify<T: Summary>(item: &T) {
    println!("{}", item.summarize());
}

// Both params must be the same concrete type:
pub fn compare<T: Summary>(a: &T, b: &T) { }

// Different concrete types allowed:
pub fn compare2(a: &impl Summary, b: &impl Summary) { }
```

### Multiple Trait Bounds with `+`

```rust
pub fn notify(item: &(impl Summary + Display)) { }

// Or with generics:
pub fn notify<T: Summary + Display>(item: &T) { }
```

### `where` Clauses

For complex bounds, `where` keeps the function signature readable:

```rust
// Messy:
fn some_fn<T: Display + Clone, U: Clone + Debug>(t: &T, u: &U) -> i32 { }

// Clean:
fn some_fn<T, U>(t: &T, u: &U) -> i32
where
    T: Display + Clone,
    U: Clone + Debug,
{ }
```

---

## Returning `impl Trait`

Return a type that implements a trait without naming the concrete type:

```rust
fn make_summarizable() -> impl Summary {
    Post {
        username: String::from("jake"),
        content: String::from("hello"),
    }
}
```

**Limitation:** You can only return one concrete type. If you need to conditionally return different types, use a trait object (`Box<dyn Trait>`) instead.

---

## Blanket Implementations

Implement a trait for every type that satisfies some bound:

```rust
impl<T: Display> ToString for T {
    // any type with Display automatically gets to_string()
}

let s = 42.to_string();  // works because i32 implements Display
```

The Rust standard library uses blanket implementations extensively.

---

## Common Standard Library Traits

| Trait | What it enables |
|---|---|
| `Display` | `{}` formatting in `println!` |
| `Debug` | `{:?}` formatting, derive with `#[derive(Debug)]` |
| `Clone` | `.clone()` — explicit deep copy |
| `Copy` | Implicit bitwise copy on assignment |
| `PartialOrd` / `Ord` | Comparison with `<`, `>`, etc. |
| `PartialEq` / `Eq` | Equality with `==`, `!=` |
| `Iterator` | `.next()` — enables `for` loops and iterator adapters |
| `From` / `Into` | Type conversions |
| `Drop` | Custom cleanup when value goes out of scope |
| `Send` / `Sync` | Thread safety markers (important for concurrency) |

---

## Related Pages

- [[rust-overview]] — full Rust topic map
- [[rust-structs]] — implementing traits on structs
- [[rust-enums-pattern-matching]] — implementing traits on enums
- [[rust-error-handling]] — `Box<dyn Error>` uses trait objects
