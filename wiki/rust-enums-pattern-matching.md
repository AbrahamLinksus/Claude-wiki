# Rust Enums and Pattern Matching

**Summary**: Enums define a type that can be one of several variants, each able to hold different data; `Option<T>` replaces null; `match` provides exhaustive, compile-time-checked branching over enum variants and other patterns.

**Pre-Req**: [[rust-structs]]

**Sources**: https://doc.rust-lang.org/book/ch06-01-defining-an-enum.html, https://doc.rust-lang.org/book/ch06-02-match.html

**Last updated**: 2026-04-16

#rust

---

## Defining Enums

An enum defines a type with a fixed set of variants. A value of that type is always exactly one of those variants.

```rust
enum Direction {
    North,
    South,
    East,
    West,
}

let heading = Direction::North;
```

Variants are namespaced with `::` under the enum name.

---

## Enum Variants with Data

Each variant can hold data — and different variants can hold different types and amounts:

```rust
enum Message {
    Quit,                        // no data
    Move { x: i32, y: i32 },    // named fields (struct-like)
    Write(String),               // single String
    ChangeColor(i32, i32, i32),  // tuple of three i32s
}
```

This is more powerful than C-style enums. The variant constructor acts like a function:

```rust
let m = Message::Write(String::from("hello"));
let mv = Message::Move { x: 10, y: 20 };
```

**IP address example — different types per variant:**

```rust
enum IpAddr {
    V4(u8, u8, u8, u8),
    V6(String),
}

let home = IpAddr::V4(127, 0, 0, 1);
let loopback = IpAddr::V6(String::from("::1"));
```

---

## Methods on Enums

Just like structs, enums can have `impl` blocks:

```rust
impl Message {
    fn call(&self) {
        // handle the message
    }
}

let m = Message::Write(String::from("hello"));
m.call();
```

---

## `Option<T>` — Replacing Null

Rust has **no null**. Instead, the standard library provides `Option<T>`:

```rust
enum Option<T> {
    Some(T),   // there is a value
    None,      // there is no value
}
```

`Option` is in the prelude — you use `Some` and `None` without any prefix.

```rust
let some_number: Option<i32> = Some(5);
let some_char: Option<char> = Some('e');
let absent: Option<i32> = None;
```

**Why this is better than null:** `Option<T>` and `T` are different types. The compiler prevents you from accidentally using an `Option<i32>` as if it were an `i32`:

```rust
let x: i32 = 5;
let y: Option<i32> = Some(5);

let sum = x + y;  // ❌ Error: can't add i32 and Option<i32>
```

You are **forced** to handle the `None` case before you can use the value. This eliminates null pointer dereferences entirely.

---

## `match` — Exhaustive Pattern Matching

`match` compares a value against a series of patterns and runs the first arm that matches. It can match any type — not just booleans like `if`.

```rust
enum Coin {
    Penny,
    Nickel,
    Dime,
    Quarter,
}

fn value_in_cents(coin: Coin) -> u8 {
    match coin {
        Coin::Penny   => 1,
        Coin::Nickel  => 5,
        Coin::Dime    => 10,
        Coin::Quarter => 25,
    }
}
```

Multi-line arms use curly braces:

```rust
match coin {
    Coin::Penny => {
        println!("Lucky penny!");
        1
    }
    Coin::Quarter => 25,
    _ => 0,
}
```

### Binding Values from Variants

Extract data from an enum variant inside a match arm:

```rust
enum Coin {
    Quarter(UsState),
    // ...
}

match coin {
    Coin::Quarter(state) => {
        println!("Quarter from {state:?}");
        25
    }
    _ => 0,
}
```

### Matching `Option<T>`

```rust
fn plus_one(x: Option<i32>) -> Option<i32> {
    match x {
        None    => None,
        Some(i) => Some(i + 1),
    }
}

let six = plus_one(Some(5));  // Some(6)
let none = plus_one(None);    // None
```

### Exhaustive Matching

`match` must cover **all possible cases**. The compiler catches missing arms:

```rust
match x {
    Some(i) => Some(i + 1),
    // ❌ Error: non-exhaustive patterns: None not covered
}
```

### Catch-All Patterns

Use a variable to capture any unmatched value:

```rust
match dice_roll {
    3 => add_fancy_hat(),
    7 => remove_fancy_hat(),
    other => move_player(other),  // binds to the value
}
```

Use `_` to match and discard (when you don't need the value):

```rust
match dice_roll {
    3 => add_fancy_hat(),
    7 => remove_fancy_hat(),
    _ => (),  // do nothing for everything else
}
```

---

## `if let` — Concise Single-Pattern Matching

When you only care about one variant, `if let` is more concise than a full `match`:

```rust
// Verbose match
match config_max {
    Some(max) => println!("Max is {max}"),
    _ => (),
}

// Equivalent if let
if let Some(max) = config_max {
    println!("Max is {max}");
}
```

`if let` can also have an `else` branch:

```rust
if let Coin::Quarter(state) = coin {
    println!("Quarter from {state:?}");
} else {
    println!("Not a quarter");
}
```

Use `match` when you need to handle multiple variants; use `if let` when only one case matters.

---

## Enums vs Structs

| | Struct | Enum |
|---|---|---|
| Use when | All fields exist simultaneously | Value is one of several options |
| Example | A `User` with name, email, age | A `Shape` that is Circle OR Square OR Triangle |
| Data | Named fields | Per-variant data |

---

## Related Pages

- [[rust-overview]] — full Rust topic map
- [[rust-structs]] — structs as the complement to enums
- [[rust-error-handling]] — `Result<T, E>` is just an enum, matched the same way
- [[rust-traits-generics]] — generic enums like `Option<T>` and `Result<T, E>` explained
