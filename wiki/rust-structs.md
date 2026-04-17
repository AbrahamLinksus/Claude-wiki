# Rust Structs

**Summary**: Structs group related named fields into a custom type; Rust supports regular structs, tuple structs (positional fields), and unit-like structs (no fields), with methods added via `impl` blocks.

**Pre-Req**: [[rust-ownership]], [[rust-borrowing-references]]

**Sources**: https://doc.rust-lang.org/book/ch05-01-defining-structs.html

**Last updated**: 2026-04-16

#rust

---

## Defining a Struct

Use the `struct` keyword followed by named fields and their types:

```rust
struct User {
    active: bool,
    username: String,
    email: String,
    sign_in_count: u64,
}
```

---

## Creating an Instance

Specify concrete values for each field using `field: value` pairs. Order doesn't matter.

```rust
let user1 = User {
    active: true,
    username: String::from("jake"),
    email: String::from("jake@example.com"),
    sign_in_count: 1,
};
```

Access fields with dot notation:

```rust
println!("{}", user1.email);
```

To modify fields, the entire instance must be declared `mut` (you cannot mark individual fields as mutable):

```rust
let mut user1 = User { /* ... */ };
user1.email = String::from("new@example.com");
```

---

## Field Init Shorthand

When function parameter names match field names, you can omit the repetition:

```rust
fn build_user(email: String, username: String) -> User {
    User {
        active: true,
        username,       // shorthand for username: username
        email,          // shorthand for email: email
        sign_in_count: 1,
    }
}
```

---

## Struct Update Syntax

Create a new instance based on an existing one, overriding specific fields with `..`:

```rust
let user2 = User {
    email: String::from("another@example.com"),
    ..user1   // copy remaining fields from user1
};
```

**Ownership note:** `..user1` moves owned fields (like `String`) from `user1`. After this, `user1.username` is no longer valid (it was moved). Fields that implement `Copy` (like `bool`, `u64`) are copied, so `user1.active` and `user1.sign_in_count` remain usable.

---

## Tuple Structs

Tuple structs have a name but no field names — just types. Useful for creating distinct types from the same underlying data.

```rust
struct Color(i32, i32, i32);
struct Point(i32, i32, i32);

let black = Color(0, 0, 0);
let origin = Point(0, 0, 0);
```

`Color` and `Point` are different types even though both hold three `i32` values — you cannot pass a `Color` where a `Point` is expected. Access values by index:

```rust
let r = black.0;
let x = origin.1;
```

Or destructure:

```rust
let Point(x, y, z) = origin;
```

---

## Unit-Like Structs

Structs with no fields, defined with just a name and semicolon:

```rust
struct AlwaysEqual;

let subject = AlwaysEqual;
```

Useful when you need to implement a trait on a type but don't need to store any data in it. Common in trait objects and marker patterns.

---

## Methods with `impl`

Add methods to a struct using an `impl` block. The first parameter is always `self` (the instance):

```rust
struct Rectangle {
    width: u32,
    height: u32,
}

impl Rectangle {
    fn area(&self) -> u32 {           // immutable borrow of self
        self.width * self.height
    }

    fn scale(&mut self, factor: u32) { // mutable borrow of self
        self.width *= factor;
        self.height *= factor;
    }
}

fn main() {
    let mut r = Rectangle { width: 10, height: 5 };
    println!("Area: {}", r.area());
    r.scale(2);
    println!("Area after scale: {}", r.area());  // 200
}
```

**Associated functions** (no `self` — like static methods, often used as constructors):

```rust
impl Rectangle {
    fn square(size: u32) -> Self {
        Self { width: size, height: size }
    }
}

let sq = Rectangle::square(10);  // called with ::
```

---

## Ownership in Structs

Structs should own their data (use `String`, not `&str`) unless you explicitly manage lifetimes. Storing references requires lifetime annotations (covered in [[rust-traits-generics]]).

```rust
struct BadUser {
    username: &str,  // ❌ Error: missing lifetime specifier
}

struct GoodUser {
    username: String,  // ✅ struct owns the data
}
```

---

## Related Pages

- [[rust-overview]] — full Rust topic map
- [[rust-enums-pattern-matching]] — enums as the complement to structs for modeling data
- [[rust-ownership]] — how struct instances interact with ownership (move, borrow)
- [[rust-traits-generics]] — implementing traits on structs, generic structs
