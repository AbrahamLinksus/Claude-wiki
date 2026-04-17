# Rust Ownership

**Summary**: Ownership is Rust's core memory management system — a set of compile-time rules that guarantee memory safety without a garbage collector by enforcing that each value has exactly one owner.

**Pre-Req**: [[rust-variables-types]]

**Sources**: https://doc.rust-lang.org/book/ch04-01-what-is-ownership.html

**Last updated**: 2026-04-16

#rust

---

## Why Ownership Exists

Most languages either use a garbage collector (which adds runtime overhead and pauses) or require manual memory management (which is error-prone). Rust's ownership system is a third path: the compiler enforces memory rules at compile time, producing programs that are both safe and as fast as C.

---

## Stack vs Heap

Understanding this is essential before ownership makes sense.

**Stack:**
- LIFO — last in, first out
- Fast — just moves a stack pointer
- Requires known, fixed size at compile time
- Integers, bools, chars, fixed arrays — all live here

**Heap:**
- Less organized, more flexible
- Slower — allocator must find free space and return a pointer
- Used when size is unknown at compile time or may grow
- `String`, `Vec`, and most collections live here

When you call a function, its local variables go on the stack. When the function returns, they're popped off. Heap memory must be explicitly managed — Rust does this through ownership.

---

## The Three Ownership Rules

1. Each value in Rust has exactly **one owner**
2. There can only be **one owner at a time**
3. When the owner goes out of scope, the value is **dropped** (freed)

```rust
{
    let s = String::from("hello");  // s is the owner
    // s is valid here
}   // s goes out of scope — drop() is called, heap memory freed
```

Rust automatically inserts the `drop()` call at the closing brace. There is no garbage collector, no `free()` — the compiler handles it.

---

## Move Semantics

For heap-allocated types, assignment **moves** ownership — the original variable is invalidated.

```rust
let s1 = String::from("hello");
let s2 = s1;   // ownership moves from s1 to s2

println!("{s1}");  // ❌ Error: s1 has been moved
println!("{s2}");  // ✅ OK
```

This prevents a **double-free error**: if both `s1` and `s2` tried to free the same heap memory when they go out of scope, it would corrupt the allocator. By invalidating `s1`, Rust ensures only one drop happens.

This is called a **move**, not a shallow copy, because the original is invalidated.

```
s1: [ ptr | len=5 | cap=5 ]  ──→  heap: "hello"
                                          ↑
s2: [ ptr | len=5 | cap=5 ]  ──→  (same heap address)
s1: (invalid after move)
```

---

## Clone: Explicit Deep Copy

To actually duplicate heap data, call `.clone()`. This is intentionally verbose — it signals that potentially expensive work is happening.

```rust
let s1 = String::from("hello");
let s2 = s1.clone();   // heap data is fully copied

println!("{s1}, {s2}");  // ✅ Both valid
```

---

## The `Copy` Trait

Types that live entirely on the stack implement the `Copy` trait. Assigning them makes a bitwise copy — the original is still valid. No move occurs.

```rust
let x = 5;
let y = x;  // copied, not moved

println!("{x}, {y}");  // ✅ Both valid
```

**Types that implement `Copy`:**
- All integer types (`i32`, `u64`, etc.)
- Floating-point types (`f32`, `f64`)
- `bool`
- `char`
- Tuples — only if all elements are `Copy` (e.g. `(i32, i32)` ✓, `(i32, String)` ✗)

**Rule:** Any type that requires heap allocation or special cleanup (i.e., implements `Drop`) cannot implement `Copy`.

---

## Ownership and Functions

Passing a value to a function **moves** or **copies** it — the same rules as assignment apply.

```rust
fn main() {
    let s = String::from("hello");
    takes_ownership(s);       // s is moved into the function
    // println!("{s}");       // ❌ s is no longer valid here

    let x = 5;
    makes_copy(x);            // x is copied (i32 is Copy)
    println!("{x}");          // ✅ x is still valid
}

fn takes_ownership(s: String) {
    println!("{s}");
}  // s is dropped here

fn makes_copy(n: i32) {
    println!("{n}");
}
```

### Return Values Transfer Ownership Back

```rust
fn gives_ownership() -> String {
    let s = String::from("yours");
    s   // ownership moves to the caller
}

fn main() {
    let s1 = gives_ownership();  // s1 is now the owner
}  // s1 is dropped here
```

This pattern (move in, operate, move out) works but is verbose. References (see [[rust-borrowing-references]]) are the practical solution.

---

## Scope and Drop Recap

```rust
let mut s = String::from("hello");
s = String::from("world");  // "hello" is dropped immediately here
// only "world" lives on
```

When a value is overwritten or goes out of scope, `drop()` is called. There is no delay, no GC pause — memory is freed exactly at the closing brace or reassignment.

---

## Related Pages

- [[rust-overview]] — full Rust topic map
- [[rust-borrowing-references]] — how to use values without transferring ownership
- [[rust-variables-types]] — which types are `Copy` and which are heap-allocated
