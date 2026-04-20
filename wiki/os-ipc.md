# OS Inter-Process Communication & Synchronization

**Summary**: The mechanisms processes use to share data and coordinate — message passing, shared memory, signals, and synchronization primitives like semaphores and mutexes.

**Pre-Req**: [[os-scheduling]] (blocking/waking is how sync primitives work), [[os-kernel-models]] (IPC is especially critical in microkernels)

**Sources**: `raw/Expanded Main Page - OSDev Wiki.md`

**Last updated**: 2026-04-18

#os

---

## Why IPC?

Processes are isolated by design (each has its own address space — see [[os-memory-management]]). IPC is the set of mechanisms the kernel provides to let them communicate despite that isolation.

---

## Message Passing

Processes exchange discrete messages via a kernel-managed channel. The kernel copies data from sender's address space to receiver's.

**Variants**:
- **Pipes**: unidirectional byte stream. `write()` → kernel buffer → `read()`. Simplest form.
- **Named pipes (FIFOs)**: like pipes but with a filesystem name, so unrelated processes can use them.
- **Message queues**: kernel-maintained queue of discrete messages with metadata (type, priority).
- **Sockets**: bidirectional, can be local (Unix sockets) or networked (TCP/UDP).

**Key property**: each message transfer involves at least one kernel copy. In a microkernel (see [[os-kernel-models]]), this overhead is paid on every cross-service call — minimizing it is a core microkernel engineering challenge.

---

## Shared Memory

The kernel maps the same physical page frames into two (or more) processes' virtual address spaces. Once set up, reads/writes between processes require zero kernel involvement — it's just memory access.

```
Process A virtual space:  [ ... | shared region | ... ]
                                        ↕ same physical frames
Process B virtual space:  [ ... | shared region | ... ]
```

**Pros**: Fastest possible IPC — no copying.

**Cons**: No isolation. A bug in one process corrupts the shared region. Requires explicit synchronization (see below) to avoid races.

---

## Signals

Signals are asynchronous notifications sent to a process. The kernel delivers them by interrupting the process and jumping to a registered signal handler.

```
kill(pid, SIGTERM)  →  kernel interrupts target  →  handler runs  →  resume or exit
```

Signals are low-bandwidth (just a number) and unreliable (same signal can't queue up). Used for process control (`SIGKILL`, `SIGSTOP`) and simple events (`SIGALRM`, `SIGSEGV`).

---

## Remote Procedure Call (RPC)

RPC makes cross-process calls look like local function calls. The caller invokes a stub that serializes arguments, sends them via message passing to a server process, waits for the reply, and returns the result as if it were a local return value.

Used heavily in microkernels and distributed systems. The "stub" generation is usually automated by an IDL (Interface Definition Language) compiler.

---

## Synchronization Primitives

Shared memory requires synchronization to prevent **race conditions** — when two threads access the same data simultaneously and at least one is writing.

### Spinlock
Busy-wait loop checking a flag until it flips. Uses `LOCK XCHG` or `CMPXCHG` (atomic CPU instructions).

```c
while (atomic_swap(&lock, 1) == 1) { /* spin */ }
// critical section
lock = 0;
```

**Use when**: critical section is very short, contention is rare, you can't sleep (e.g., interrupt handlers). **Never** hold a spinlock across a sleep or blocking call.

### Mutex (Mutual Exclusion Lock)
Like a spinlock but puts the waiter to sleep (blocks on [[os-scheduling|wait queue]]) instead of spinning. The OS wakes it when the lock is released. Much better for longer critical sections or when contention is high.

### Semaphore
A counter-based generalization. `wait()` (P) decrements; if it hits 0, blocks. `signal()` (V) increments; if threads are waiting, wakes one.

- Binary semaphore (count = 0 or 1) is equivalent to a mutex.
- Counting semaphore allows N concurrent accessors — useful for resource pools.

### Condition Variable
Used with a mutex. A thread acquires the mutex, checks a condition, and if not satisfied calls `wait()` — which atomically releases the mutex and sleeps. Another thread signals when the condition becomes true, waking the waiter.

Classic pattern: producer-consumer with a bounded buffer.

### Deadlock

Four necessary conditions (Coffman conditions):
1. **Mutual exclusion**: resources can't be shared.
2. **Hold and wait**: thread holds one resource while waiting for another.
3. **No preemption**: resources can't be forcibly taken.
4. **Circular wait**: A waits for B, B waits for A.

Break any one condition to prevent deadlock. Most OSes use lock ordering (always acquire locks in the same global order) to break circular wait.

---

## Summary: When to Use What

| Mechanism | Best for |
|---|---|
| Pipe | Parent/child, streaming data |
| Message queue | Decoupled producers/consumers |
| Shared memory | High-bandwidth, latency-critical data sharing |
| Signal | Asynchronous process events |
| RPC | Microkernel services, distributed systems |
| Spinlock | ISR-safe, very short critical sections |
| Mutex | General kernel/user synchronization |
| Semaphore | Resource pools, ordering guarantees |
| Condition variable | Waiting on state changes |
