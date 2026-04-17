# Rust Borrowing and References

**Summary**: References let you use a value without taking ownership; Rust enforces strict rules — at any point you can have either one mutable reference or any number of immutable references, never both.

**Pre-Req**: [[rust-ownership]]

**Sources**: https://doc.rust-lang.org/book/ch04-02-references-and-borrowing.html

**Last updated**: 2026-04-16

#rust

---

## What is a Reference?

A reference is a pointer to a value owned by someone else. Creating a reference is called **borrowing**. You borrow the value, use it, and return it — you never own it and never drop it.

Use `&` to create a reference:

```rust
fn main() {
    let s1 = String::from("hello");
    let len = calculate_length(&s1);  // pass a reference
    println!("{s1} has length {len}"); // s1 is still valid
}

fn calculate_length(s: &String) -> usize {
    s.len()
}  // s goes out of scope, but since it doesn't own the String, nothing is dropped
```

Without references, you'd have to move `s1` into the function and return it back to use it again. References solve this verbosity cleanly.

---

## Immutable References (Default)

By default, references are immutable — you cannot modify the value through them:

```rust
fn change(s: &String) {
    s.push_str(", world");  // ❌ Error: cannot borrow as mutable
}
```

---

## Mutable References

Add `mut` to both the variable and the reference to allow modification:

```rust
fn main() {
    let mut s = String::from("hello");
    change(&mut s);
    println!("{s}");  // "hello, world"
}

fn change(s: &mut String) {
    s.push_str(", world");  // ✅ OK
}
```

---

## The Borrowing Rules

These are the two non-negotiable rules enforced at compile time:

**Rule 1:** At any given time, you can have **either**:
- Any number of immutable references (`&T`), **or**
- Exactly **one** mutable reference (`&mut T`)

**Rule 2:** References must always be **valid** (no dangling references).

---

### Rule 1: No Simultaneous Mutable References

```rust
let mut s = String::from("hello");

let r1 = &mut s;
let r2 = &mut s;  // ❌ Error: cannot borrow s as mutable more than once

println!("{r1}, {r2}");
```

**Why:** Two mutable references could create a data race — one reference changes data while the other reads it, leading to undefined behavior. Rust prevents this entirely at compile time.

Use a new scope to work around this:

```rust
let mut s = String::from("hello");
{
    let r1 = &mut s;
    // use r1
}  // r1 goes out of scope here

let r2 = &mut s;  // ✅ OK — r1 is gone
```

### No Mutable Reference While Immutable References Exist

```rust
let mut s = String::from("hello");

let r1 = &s;     // OK
let r2 = &s;     // OK — multiple immutable refs fine
let r3 = &mut s; // ❌ Error: cannot borrow as mutable while immutable refs exist

println!("{r1}, {r2}, {r3}");
```

But this **is** valid — Rust tracks the last use of each reference:

```rust
let mut s = String::from("hello");

let r1 = &s;
let r2 = &s;
println!("{r1} and {r2}");  // last use of r1 and r2

let r3 = &mut s;  // ✅ OK — r1 and r2 are no longer used after this point
println!("{r3}");
```

A reference's scope ends at its **last use**, not at the end of the block. This is called **Non-Lexical Lifetimes (NLL)**.

---

### Rule 2: No Dangling References

Rust prevents you from returning a reference to a value that will be dropped:

```rust
fn dangle() -> &String {
    let s = String::from("hello");
    &s          // ❌ Error: s will be dropped when this function returns
}
```

**Fix:** Return the owned value, transferring ownership to the caller:

```rust
fn no_dangle() -> String {
    let s = String::from("hello");
    s   // ownership moves to caller — no dangling
}
```

---

## Dereferencing

The `*` operator dereferences a reference — it follows the pointer to get to the actual value:

```rust
let x = 5;
let r = &x;

println!("{}", *r);  // dereference to get 5
```

In most cases Rust auto-dereferences for you (e.g., calling methods on references), so explicit `*` is mainly needed with raw pointers in `unsafe` code.

---

## Summary: Reference Rules at a Glance

| Situation | Allowed? |
|---|---|
| Multiple immutable refs (`&T`) | ✅ Yes |
| One mutable ref (`&mut T`) | ✅ Yes |
| Mutable ref + any other ref | ❌ No |
| Two mutable refs | ❌ No |
| Reference outliving its value | ❌ No |

---

## Related Pages

- [[rust-overview]] — full Rust topic map
- [[rust-ownership]] — ownership rules that references work alongside
- [[rust-traits-generics]] — lifetimes (the advanced extension of these rules)
