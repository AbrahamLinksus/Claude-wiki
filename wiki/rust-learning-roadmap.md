---
# Rust Learning Roadmap

**Summary**: A focused 2-day sprint to learn core Rust before starting OS development, covering ownership, traits, unsafe Rust, and `no_std`.

**Sources**: https://doc.rust-lang.org/book/, https://os.phil-opp.com/

**Last updated**: 2026-04-16

---

## Goal

Get through enough Rust to start [Writing an OS in Rust](https://os.phil-opp.com/) by end of weekend (2026-04-19).

You will not *master* Rust in 2 days — the goal is functional literacy: understand the ownership model, be able to read and write basic Rust, and know enough unsafe Rust to not be lost in kernel code.

Your C++ background will help — many concepts (pointers, stack vs heap, zero-cost abstractions) carry over directly.

---

## Day 1 — Thursday 2026-04-16: The Core Language

Work through [The Rust Book](https://doc.rust-lang.org/book/) chapters in order. Do not skip.

| Chapters | Topics | Why it matters |
|---|---|---|
| Ch 1–2 | Setup, guessing game | Get Rust running, see the syntax |
| Ch 3 | Variables, types, functions, control flow | Syntax fundamentals |
| **Ch 4** | **Ownership, borrowing, slices** | **The most important chapter. Rust's core idea.** |
| **Ch 5** | **Structs** | How you define data types |
| **Ch 6** | **Enums + Pattern matching** | `Option`, `Result` — used everywhere |

**End of Day 1 checkpoint**: You understand why Rust's borrow checker exists and can write a program that compiles without fighting it blindly.

---

## Day 2 — Friday 2026-04-17: Systems-Level Rust

| Chapters | Topics | Why it matters |
|---|---|---|
| Ch 7 | Modules, packages, crates | How Rust code is organized |
| Ch 8 | Collections (Vec, HashMap) | Standard data structures |
| **Ch 9** | **Error handling** | `Result<T, E>`, `?` operator — kernel code uses this constantly |
| **Ch 10** | **Generics, Traits, Lifetimes** | Rust's abstraction system |
| Ch 13 | Iterators and closures | Idiomatic Rust patterns |
| **Ch 19** | **Unsafe Rust** | Raw pointers, `unsafe` blocks — required for OS/hardware work |

After Ch 19, skim:
- [`no_std` documentation](https://docs.rust-embedded.org/book/intro/no-std.html) — OS kernels can't use the standard library
- Inline assembly in Rust (`core::arch::asm!`) — you'll need this for interrupt handlers

**End of Day 2 checkpoint**: You can write unsafe Rust, understand why it exists, and know what `no_std` means.

---

## Weekend — Start the OS Tutorial

[Writing an OS in Rust by Philipp Oppermann](https://os.phil-opp.com/)

Go through in order:
1. A Freestanding Rust Binary
2. A Minimal Rust Kernel
3. VGA Text Mode
4. CPU Exceptions
5. Double Faults

Do not rush these. The OS concepts (paging, interrupts, GDT) are harder than the Rust at this point.

---

## Key Concepts to Internalize

### Ownership (Day 1 priority)
- Every value has one owner
- When the owner goes out of scope, the value is dropped
- You can borrow (`&T`) or mutably borrow (`&mut T`), but not both at once
- This replaces both GC and manual `free()` — [[cpp-fundamental-types]] context: Rust's `i32` is like C++'s `int32_t` but with ownership rules on top

### Traits vs C++ Concepts
- Rust traits ≈ C++ abstract base classes / concepts
- `impl Trait for Type` adds behavior to a type
- Common OS-relevant traits: `Copy`, `Send`, `Sync`, `Drop`

### Unsafe Rust
- The `unsafe` keyword unlocks: raw pointers, calling unsafe functions, accessing mutable statics, implementing unsafe traits
- Kernel code lives mostly in `unsafe` blocks — memory-mapped I/O, interrupt tables, page tables all require it
- Unsafe doesn't mean "ignore Rust" — it means "I am taking responsibility for invariants the compiler can't check"

### `no_std`
- Standard library (`std`) assumes OS primitives (heap, threads, files)
- Kernels use `core` (language primitives only) and optionally `alloc` (heap, once you set one up)
- Add `#![no_std]` at the top of your kernel crate

---

## What to Skip (For Now)

- Ch 14: Cargo workspaces — not needed yet
- Ch 15: Smart pointers (`Box`, `Rc`, `RefCell`) — useful later, skip for now
- Ch 16: Concurrency — come back when implementing a scheduler
- Ch 17: OOP patterns — not relevant for OS work
- Ch 18: Patterns — skim only

---

## Resources

| Resource | When to use |
|---|---|
| [The Rust Book](https://doc.rust-lang.org/book/) | Primary learning source |
| [Rustlings](https://github.com/rust-lang/rustlings) | Exercises to reinforce chapters |
| [Rust by Example](https://doc.rust-lang.org/rust-by-example/) | Quick syntax lookup |
| [Writing an OS in Rust](https://os.phil-opp.com/) | OS tutorial — start after Day 2 |
| [Redox OS](https://gitlab.redox-os.org/redox-os/kernel) | Real Rust kernel to read |
| [Rust Embedded Book](https://docs.rust-embedded.org/book/) | `no_std` and hardware interaction |
