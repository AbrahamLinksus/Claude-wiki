# OS Scheduling

**Summary**: How an OS manages processes and threads — their data structures, context switching mechanics, and scheduling algorithms that decide who runs next.

**Pre-Req**: [[os-interrupts]] (the timer interrupt drives preemption), [[os-memory-management]] (each process has its own page table)

**Sources**: `raw/Expanded Main Page - OSDev Wiki.md`

**Last updated**: 2026-04-18

#os

---

## Processes vs Threads

| | Process | Thread |
|---|---|---|
| Address space | Own virtual address space | Shares address space with siblings |
| Resources | Own file descriptors, handles | Shared with process |
| Switch cost | High (page table swap + TLB flush) | Low (register state only) |
| Isolation | Strong | Weak — a bug in one thread can corrupt others |

A **process** is the unit of isolation. A **thread** is the unit of scheduling. Every process has at least one thread.

---

## Process Control Block (PCB)

The kernel represents each process with a PCB (also called task struct). Minimum fields:

```c
struct process {
    uint32_t pid;
    enum { RUNNING, READY, BLOCKED, ZOMBIE } state;
    uintptr_t page_table;    // CR3 value
    struct registers context; // saved registers
    void *kernel_stack;
    // file descriptors, signal handlers, etc.
};
```

---

## Context Switching

A context switch saves the current thread's register state and restores another's. On x86:

1. Timer interrupt fires (or process voluntarily yields).
2. CPU pushes `SS`, `RSP`, `RFLAGS`, `CS`, `RIP` onto the kernel stack automatically (interrupt frame).
3. Your ISR saves the remaining general-purpose registers.
4. Scheduler picks the next thread.
5. If crossing processes: load new `CR3` (page table swap).
6. Restore the new thread's registers, `iretq` back to user mode.

The key invariant: every register the interrupted code "owns" must be saved and restored exactly.

```
Thread A running
    → timer IRQ fires
    → save A's registers
    → scheduler picks B
    → load B's registers (+ CR3 if different process)
    → iretq → Thread B running
```

---

## Scheduling Algorithms

### First-Come First-Served (FCFS)
Run each process to completion in arrival order. Simple, but long jobs starve short ones. Not used in interactive systems.

### Round Robin
Each process gets a fixed **time slice** (quantum, e.g. 10ms). When the quantum expires, the next ready process runs. The timer interrupt is the preemption mechanism (see [[os-interrupts]]).

### Priority Scheduling
Each process has a priority. Highest-priority ready process always runs. Risk: **starvation** — low-priority processes never run if high-priority ones keep arriving. Fix: **aging** (raise priority over time).

### Multilevel Feedback Queue (MLFQ)
Used by Linux and BSD. Multiple queues with different priorities and time slices. Processes move between queues based on behavior — CPU-bound processes drop in priority; interactive processes rise. Approximates "favor I/O-bound and interactive tasks" without requiring prior knowledge.

### Completely Fair Scheduler (CFS)
Linux's current scheduler. Tracks **virtual runtime** (time already given, weighted by priority/niceness). Always schedules the process with the lowest virtual runtime. Uses a red-black tree keyed by virtual runtime for O(log n) selection.

### Multiprocessor Scheduling
On SMP systems (multiple CPUs), each CPU has a local run queue. **Load balancing** periodically migrates processes between queues to prevent idle CPUs. Cache affinity matters: migrating a process from CPU A to CPU B invalidates CPU B's cache of that process's data.

---

## Blocking and Sleeping

A process blocks when it must wait for an event (I/O completion, lock, timer). The scheduler moves it to a **wait queue** associated with that event. When the event fires, the kernel moves the process back to the ready queue.

```
Running → (read() called, disk not ready) → Blocked
                                                ↓ (disk IRQ fires)
                                             Ready → Running
```

This is why you never busy-wait in kernel code — it wastes CPU time. Use wait queues and block instead.

---

## Kernel vs User Threads

- **User threads** (green threads): scheduled by a runtime in user space. Fast to create/switch, but a blocking syscall blocks the whole process.
- **Kernel threads**: scheduled by the OS. Blocking one doesn't block others. More expensive.
- **M:N threading**: M user threads multiplexed over N kernel threads. Complex but used by some runtimes (Go goroutines, early Java).
