# OS Kernel Models

**Summary**: The four major kernel architecture patterns — monolithic, microkernel, exokernel, and modular — their tradeoffs, and when to choose each.

**Pre-Req**: [[os-overview]], [[os-ipc]] (microkernel relies heavily on IPC)

**Sources**: `raw/Expanded Main Page - OSDev Wiki.md`

**Last updated**: 2026-04-18

#os

---

## Overview

The kernel model is the most fundamental design decision you make. It determines where code runs (kernel space vs user space), how subsystems communicate, and how failures propagate.

---

## Monolithic Kernel

All OS services (scheduler, file system, drivers, memory manager) run in kernel space as a single large program. A function call between subsystems is just a function call — no IPC overhead.

**Pros**: Simple to implement, fast (no context switches for IPC), well-understood (Linux, BSD, early Unix).

**Cons**: A bug in any driver crashes the whole kernel. Harder to isolate faults. Grows large and complex over time.

**Examples**: Linux, FreeBSD, Windows NT (largely).

**Verdict for OS dev beginners**: Start here. The simplicity outweighs the downsides at hobby scale.

---

## Microkernel

Only the absolute minimum runs in kernel space: IPC, basic memory management, and scheduling. Everything else (drivers, file systems, networking) runs as user-space servers and communicates via [[os-ipc|message passing]].

```
User space:  [File Server] [Driver] [Network Stack]
                   ↕ IPC        ↕ IPC       ↕ IPC
Kernel space: [ IPC | Memory | Scheduler ]
```

**Pros**: A crashed driver doesn't take down the kernel. Easier to verify formally. Components are replaceable.

**Cons**: IPC overhead on every cross-component call. Harder to implement correctly. Performance is a constant battle.

**Examples**: MINIX 3, seL4, QNX, GNU Hurd.

---

## Exokernel

The kernel does almost nothing except multiplex hardware securely between applications. There's no abstraction layer — apps get raw access to disk blocks, CPU time slices, and memory frames. Each application (or a library OS) builds its own abstractions on top.

**Pros**: Applications can optimize their own resource use without kernel overhead. Maximum flexibility.

**Cons**: Extremely hard to build and use. The burden of abstraction shifts to every application. Very experimental.

**Examples**: MIT Exokernel (research), Aegis.

---

## Modular Kernel

A hybrid: the kernel has a monolithic core but loads subsystems as runtime modules that can be inserted/removed without rebooting.

**Pros**: Flexibility of a microkernel with performance closer to monolithic. Modules run in kernel space so no IPC overhead.

**Cons**: A buggy module can still crash the kernel. Module interface versioning is painful.

**Examples**: Linux (with loadable kernel modules), macOS XNU.

---

## Comparison Table

| Model | Where services run | IPC needed? | Fault isolation | Perf overhead |
|---|---|---|---|---|
| Monolithic | Kernel space | No | Low | None |
| Microkernel | User space | Yes | High | High |
| Exokernel | App/library | No | Medium | Low |
| Modular | Kernel space (loadable) | No | Medium | None |

---

## Task Models

Independent of kernel architecture, you also choose a task model:

- **Monotasking**: One process runs at a time (DOS-style). Simplest possible scheduler.
- **Multitasking**: Multiple processes share the CPU via [[os-scheduling|scheduling]].
- **Real-Time**: Scheduling guarantees worst-case response times (RTOS). Used in embedded/safety systems.

See [[os-scheduling]] for how to implement these.
