# Wiki Index

Personal knowledge base — maintained by Claude Code.

---

## Rust — OS Development Track

- [[rust-overview]] — Full Rust topic map, why Rust exists, comparison with C/C++/Go
- [[rust-learning-roadmap]] — 2-day sprint plan to learn Rust core before starting OS dev
- [[rust-variables-types]] — Variables, mutability, shadowing, constants, scalar and compound types
- [[rust-functions-control-flow]] — Function syntax, statements vs expressions, `if`, `loop`, `while`, `for`
- [[rust-ownership]] — Ownership rules, move semantics, `Copy` trait, stack vs heap
- [[rust-borrowing-references]] — References, borrowing rules, mutable references, no dangling pointers
- [[rust-structs]] — Struct definition, field init shorthand, update syntax, tuple structs, `impl` blocks
- [[rust-enums-pattern-matching]] — Enums, `Option<T>`, `match`, exhaustive patterns, `if let`
- [[rust-error-handling]] — `Result<T, E>`, `?` operator, `unwrap`, `expect`, `panic!`
- [[rust-traits-generics]] — Traits, default implementations, generics, trait bounds, monomorphization
- [[rust-macros]] — Metaprogramming in Rust: how macros work, `println!` explained, common macros
- [[rust-strings]] — `&str` vs `String`, UTF-8 rules, why integer indexing is banned, slicing, iteration, common methods

---

## C++ — Learning from Scratch

- [[cpp-introduction]] — C++ overview and learning roadmap (#program)
- [[cpp-program-structure]] — How a C++ program is structured and compiled (#program)
- [[cpp-fundamental-types]] — All built-in types: integers, floats, bool, char, void (#program)
- [[cpp-declarations-definitions]] — Declarations vs definitions, qualifiers, storage classes (#program)

---

---

## Agentic AI

- [[agentic-ai-overview]] — Full topic map for agentic AI: agent loop, tools, planning, memory, multi-agent, eval, safety (#agentic)
- [[agentic-agent-loop]] — The observe → think → act cycle; loop variants, stopping conditions, failure modes (#agentic)
- [[agentic-multi-agent]] — Pipeline pattern: planner → parallel workers → reviewer → aggregated output (#agentic)
- [[agentic-langchain]] — LangChain & LangGraph: framework abstractions for chains, tools, memory, and multi-agent state machines (#agentic)

---

## OS Development

- [[os-overview]] — Full topic map and recommended learning order for building an OS from scratch (#os)
- [[os-toolchain]] — GCC cross-compiler, NASM, LD linker scripts, QEMU/Bochs emulators (#os)
- [[os-x86-modes]] — Real Mode → Protected Mode → Long Mode (x86-64), CPU registers reference (#os)
- [[os-boot-sequence]] — BIOS/UEFI → GRUB/Limine → kernel entry, GDT, TSS, A20 line (#os)
- [[os-interrupts]] — IDT, ISRs, exceptions, PIC, APIC, NMI, timer, syscall interface (#os)
- [[os-kernel-models]] — Monolithic vs microkernel vs exokernel vs modular; task models (#os)
- [[os-memory-management]] — Segmentation, paging, page frame allocator, heap, MMU, virtual address layout (#os)
- [[os-scheduling]] — Processes vs threads, PCB, context switching, scheduling algorithms, SMP (#os)
- [[os-ipc]] — Message passing, shared memory, signals, RPC, semaphores, mutexes, deadlock (#os)
- [[os-hardware]] — I/O ports, MMIO, storage (ATA/AHCI/NVMe), video, input, PCI, ACPI (#os)

---

**Last updated**: 2026-04-18
