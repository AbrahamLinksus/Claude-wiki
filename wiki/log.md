# Wiki Operation Log

Append-only record of all wiki operations.

---

## 2026-04-16 — Session start: C++ from scratch

**Source**: https://en.cppreference.com/w/cpp (devdocs.io/cpp mirrors this)

**Tag**: #program

**Pages created**:
- `cpp-introduction.md` — What C++ is, why it matters for systems dev, compilation pipeline, learning roadmap
- `cpp-program-structure.md` — Translation units, main(), scope, name lookup, storage duration, linkage, ODR, header guards
- `cpp-fundamental-types.md` — All built-in types: integers (signed/unsigned, fixed-width), floats (IEEE-754), bool, char variants, void, nullptr
- `cpp-declarations-definitions.md` — Declaration vs definition, ODR, const/volatile qualifiers, auto/static/extern/thread_local/mutable storage class specifiers

**Index updated**: yes

---

## 2026-04-16 — Rust OS development learning roadmap

**Source**: https://doc.rust-lang.org/book/, https://os.phil-opp.com/

**Pages created**:
- `rust-learning-roadmap.md` — 2-day sprint plan covering ownership, traits, unsafe Rust, no_std, and the phil-opp OS tutorial

**Index updated**: yes

---

## 2026-04-16 — Rust Macros

**Source**: `oWiki/Rust from start.md`

**Pages created**:
- `rust-macros.md` — What macros are, why they exist, how `println!` works, common macros reference table

**Index updated**: yes

---

## 2026-04-16 — Full Rust Knowledge Base

**Source**: https://doc.rust-lang.org/book/ (official Rust Book, chapters 3–10)

**Tag**: #rust

**Pages created**:
- `rust-overview.md` — Summary page: topic map, why Rust exists, comparison table with C/C++/Go, recommended learning order
- `rust-variables-types.md` — Variables, mutability, shadowing, constants, all scalar types (integers, floats, bool, char), compound types (tuples, arrays), overflow behavior
- `rust-functions-control-flow.md` — Function syntax, snake_case convention, parameters, statements vs expressions, implicit returns, `if` as expression, `loop`/`while`/`for`, loop labels, range iteration
- `rust-ownership.md` — Stack vs heap, three ownership rules, move semantics, clone, `Copy` trait, scope and drop, ownership through functions
- `rust-borrowing-references.md` — References (`&T`, `&mut T`), borrowing rules, NLL (non-lexical lifetimes), dangling reference prevention, dereferencing
- `rust-structs.md` — Struct definition, field init shorthand, struct update syntax, tuple structs, unit-like structs, `impl` blocks, methods, associated functions
- `rust-enums-pattern-matching.md` — Enum definition, variants with data, methods on enums, `Option<T>` replacing null, `match` expressions, exhaustive matching, catch-all patterns, `if let`
- `rust-error-handling.md` — `Result<T, E>`, `match` on Result, `unwrap`, `expect`, `?` operator, error propagation, `panic!`, when to use each
- `rust-traits-generics.md` — Generic functions/structs/enums/methods, monomorphization (zero-cost), trait definition, implementing traits, orphan rule, default implementations, trait bounds (`+`, `where`), `impl Trait`, blanket implementations, common stdlib traits

**Index updated**: yes
