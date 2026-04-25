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

---

## 2026-04-18 — Rust Strings

**Source**: https://doc.rust-lang.org/book/ch08-02-strings.html

**Tag**: #rust

**Pages created**:
- `rust-strings.md` — `&str` vs `String`, UTF-8 encoding, why integer indexing is disallowed, slicing, `.chars()` vs `.bytes()`, common methods, coercion rules

**Index updated**: yes

---

## 2026-04-18 — OS Development Knowledge Cluster

**Source**: `raw/Expanded Main Page - OSDev Wiki.md`

**Tag**: #os

**Pages created**:
- `os-overview.md` — Summary/topic map: learning order, all concept links, key decision points (language, kernel model, bootloader)
- `os-toolchain.md` — GCC cross-compiler (x86_64-elf target), NASM/GAS/FASM, LD linker scripts, QEMU/Bochs emulator setup and debugging
- `os-x86-modes.md` — Real Mode, Protected Mode (rings, GDT), Long Mode (x86-64), mode transition code, CPU register reference table
- `os-boot-sequence.md` — BIOS vs UEFI, MBR/GPT, GRUB/Limine protocols, GDT/TSS setup, A20 line, early kernel init checklist
- `os-interrupts.md` — IDT, gate types, exception vectors (PF, GPF, DF), PIC remapping, APIC/LAPIC/IO-APIC, NMI, ISR rules, syscall (int 0x80 vs SYSCALL MSR)
- `os-kernel-models.md` — Monolithic vs microkernel vs exokernel vs modular, comparison table, task models (mono/multi/RT)
- `os-memory-management.md` — Segmentation, 4-level paging, TLB, page frame allocators (bitmap/stack/buddy), heap allocators (bump/free-list/slab), MMU registers, virtual address layout
- `os-scheduling.md` — Process vs thread, PCB fields, context switch mechanics (iretq), FCFS/round-robin/priority/MLFQ/CFS algorithms, SMP load balancing, blocking/wait queues
- `os-ipc.md` — Message passing (pipes/queues/sockets), shared memory, signals, RPC, spinlock/mutex/semaphore/condvar, deadlock (Coffman conditions)
- `os-hardware.md` — I/O ports vs MMIO, PIT/APIC timer/HPET, ATA/AHCI/NVMe storage, VGA text mode/linear framebuffer, PS/2 keyboard+mouse, PCI enumeration, ACPI tables (RSDP/MADT/MCFG)

**Index updated**: yes

---

## 2026-04-25 — GitHub Copilot Certification (GH-300)

**Source**: https://learn.microsoft.com/credentials/certifications/resources/study-guides/gh-300 (web, no raw file)

**Pages created**:
- `github-copilot-certification.md` — Summary page: exam details, domain weights, recommended study order, official resources
- `copilot-responsible-ai.md` — Risks and limitations of GenAI, ethical usage, harm mitigation, validating AI output
- `copilot-features.md` — Copilot in IDE, CLI, Agent Mode, Edit Mode, MCP, Sub-Agents, Spaces, Spark, PR summaries, org-wide management
- `copilot-data-architecture.md` — Data flow and sharing, prompt construction, proxy filtering, post-processing, suggestion lifecycle, LLM limits
- `copilot-prompt-engineering.md` — Prompt structure, context determination, zero-shot/few-shot, best practices, chat history, prompt file reuse
- `copilot-productivity.md` — Code gen, refactoring, documentation, sample data, legacy modernization, unit/integration tests, security, performance
- `copilot-privacy-safeguards.md` — Content exclusions, editor settings, output ownership, duplication detection, security warnings, troubleshooting

**Index updated**: yes

---

## 2026-04-18 — Agentic AI knowledge cluster (session start)

**Source**: Session-built (no raw file yet)

**Tag**: #agentic

**Pages created**:
- `agentic-ai-overview.md` — Summary/topic map page: all concept links, recommended reading order, roadmap
- `agentic-agent-loop.md` — The core agent loop pattern, ReAct/Reflexion/Plan-execute variants, failure modes
- `agentic-multi-agent.md` — Full pipeline: goal → planner → parallel workers → reviewer → final output, failure modes table
- `agentic-langchain.md` — LangChain & LangGraph overview: chains, agents, tools, memory, state machine orchestration, vs rolling your own

**Index updated**: yes
