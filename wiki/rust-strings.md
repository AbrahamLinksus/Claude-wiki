# Rust Strings

**Summary**: Rust has two main string types — `&str` (borrowed slice) and `String` (owned heap allocation) — both enforcing UTF-8, which prevents direct integer indexing and makes string handling more explicit than in most languages.

**Pre-Req**: [[rust-ownership]], [[rust-borrowing-references]]

**Sources**: https://doc.rust-lang.org/book/ch08-02-strings.html

**Last updated**: 2026-04-18

---

#rust

## The Two String Types

Rust has two string types that you'll use constantly:

| | `&str` | `String` |
|---|---|---|
| What it is | Borrowed reference to UTF-8 bytes | Owned, heap-allocated UTF-8 string |
| Size | Fixed (known at compile or runtime) | Growable |
| Stored | Usually stack (pointer + length) | Heap data, pointer on stack |
| Mutability | Immutable | Mutable (with `mut`) |
| Allocation | No allocation | Allocates on heap |

Think of `&str` as a *view* into some string data, and `String` as *owning* that data.

---

## String Literals — `&str`

String literals in your source code are `&str`:

```rust
let s = "hello";        // type: &str
let s: &str = "hello";  // explicit annotation
```

The text `"hello"` is baked into the binary. `s` is a reference pointing into that read-only memory — a borrowed slice with a `'static` lifetime (lives as long as the program).

String literals are immutable. You cannot call `.push_str()` on a `&str`.

---

## `String` — Owned, Growable

Use `String` when you need to own or modify string data.

### Creating a `String`

```rust
let s = String::new();              // empty string
let s = String::from("hello");      // from a literal
let s = "hello".to_string();        // same result, alternate syntax
```

### Growing a `String`

```rust
let mut s = String::from("hello");

s.push_str(", world");  // append a &str — does not take ownership
s.push('!');            // append a single char

println!("{s}");        // "hello, world!"
```

`push_str` takes a `&str`, not a `String`, so the argument is not consumed:

```rust
let mut s1 = String::from("hello");
let s2 = String::from(", world");

s1.push_str(&s2);  // s2 is borrowed, still usable after this
println!("{s2}");  // ✅ still valid
```

### Concatenation with `+`

```rust
let s1 = String::from("hello");
let s2 = String::from(", world");
let s3 = s1 + &s2;  // s1 is MOVED here, s2 is borrowed
// println!("{s1}");  ❌ s1 was moved
println!("{s3}");    // "hello, world"
```

The `+` operator calls `fn add(self, s: &str) -> String`. It takes ownership of `s1` (efficiency — reuses its allocation) and borrows `s2`. This gets unwieldy with multiple strings.

### `format!` — Concatenation Without Moving

For combining multiple strings, `format!` is cleaner and doesn't take ownership of anything:

```rust
let s1 = String::from("tic");
let s2 = String::from("tac");
let s3 = String::from("toe");

let s = format!("{s1}-{s2}-{s3}");  // all three still valid
println!("{s1}");  // ✅
```

---

## UTF-8 and Why You Can't Index Strings

Rust strings are **always valid UTF-8**. This is a hard guarantee — unlike C/C++, you can never have a `String` with invalid encoding.

The consequence: **you cannot index a string by integer**:

```rust
let s = String::from("hello");
let c = s[0];  // ❌ compile error: String cannot be indexed by integer
```

Why? Because in UTF-8, characters are 1–4 bytes wide. Index `0` would return a byte (`u8`), not a character — and returning a partial character would be meaningless or misleading.

Consider:
```rust
let s = String::from("Здравствуйте");  // Russian — each char is 2 bytes
```

`s[0]` would return `208` (first byte of `З`), not `З`. Rust refuses to do this silently.

### String Length

`.len()` returns the number of **bytes**, not characters:

```rust
let hello = String::from("hello");
let zdravo = String::from("Здравствуйте");

println!("{}", hello.len());   // 5 (5 bytes, 5 chars — ASCII)
println!("{}", zdravo.len());  // 24 (24 bytes, 12 Cyrillic chars)
```

---

## Slicing Strings

You *can* slice strings using byte ranges, but it will **panic at runtime** if you slice in the middle of a multi-byte character:

```rust
let s = "Здравствуйте";
let slice = &s[0..4];   // ✅ "Зд" — each Cyrillic char is 2 bytes, so 4 bytes = 2 chars
let bad   = &s[0..1];   // ❌ panics: byte index 1 is inside a char boundary
```

Use slicing carefully. When working with ASCII-only data it's safe; for Unicode, prefer iterating.

---

## Iterating Over Strings

### By character — `.chars()`

Returns Unicode scalar values (`char`):

```rust
for c in "Зд".chars() {
    println!("{c}");
    // З
    // д
}
```

### By byte — `.bytes()`

Returns raw `u8` bytes:

```rust
for b in "hello".bytes() {
    println!("{b}");
    // 104, 101, 108, 108, 111
}
```

Use `.chars()` for text processing. Use `.bytes()` for binary/network protocols.

### Counting actual characters

```rust
let s = "Здравствуйте";
println!("{}", s.chars().count());  // 12 — actual character count
```

---

## Common String Methods

```rust
let s = String::from("  Hello, World!  ");

s.len()                      // byte length
s.is_empty()                 // true if len == 0
s.contains("World")          // true
s.starts_with("  Hello")     // true
s.ends_with("!  ")           // true
s.to_lowercase()             // "  hello, world!  "
s.to_uppercase()             // "  HELLO, WORLD!  "
s.trim()                     // "Hello, World!" — strips leading/trailing whitespace
s.trim_start()               // strips leading only
s.trim_end()                 // strips trailing only
s.replace("World", "Rust")   // "  Hello, Rust!  "
```

Splitting:
```rust
let csv = "one,two,three";

for part in csv.split(',') {
    println!("{part}");  // one / two / three
}

let parts: Vec<&str> = csv.split(',').collect();
```

Finding:
```rust
let s = "hello world";
s.find('o')           // Some(4) — byte index of first match
s.find("world")       // Some(6)
s.find('z')           // None
```

---

## Converting Between `&str` and `String`

```rust
// &str → String
let owned = "hello".to_string();
let owned = String::from("hello");

// String → &str (borrowing)
let s = String::from("hello");
let slice: &str = &s;       // coercion: &String → &str
let slice: &str = &s[..];   // explicit full slice — same result
```

The coercion from `&String` to `&str` happens automatically in most contexts — if a function takes `&str`, you can pass `&String` directly:

```rust
fn greet(name: &str) {
    println!("Hello, {name}!");
}

let s = String::from("Alice");
greet(&s);       // ✅ &String coerces to &str automatically
greet("Bob");    // ✅ &str literal works too
```

**Rule of thumb**: if your function only needs to *read* a string, take `&str`. It accepts both `&String` and string literals, making the function more flexible.

---

## Summary: When to Use Which

| Situation | Use |
|---|---|
| Reading/borrowing string data (function params) | `&str` |
| Owning string data, building strings at runtime | `String` |
| String constant in source code | `&str` (literal) |
| Returning a new string from a function | `String` |
| Passing a string to a function that just reads it | `&str` |

---

## Related Pages

- [[rust-ownership]] — move semantics, why `+` consumes `s1`
- [[rust-borrowing-references]] — why `&str` is a borrow, lifetime of string slices
- [[rust-variables-types]] — scalar types, `char` type, compound types overview
- [[rust-collections]] — `Vec<T>`, `HashMap` — the other main heap-allocated collections
