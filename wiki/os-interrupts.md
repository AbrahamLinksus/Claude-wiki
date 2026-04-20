# OS Interrupts

**Summary**: How the CPU and OS handle interrupts and exceptions — the IDT, ISRs, the PIC/APIC hierarchy, and how the timer interrupt drives preemptive scheduling.

**Pre-Req**: [[os-x86-modes]] (interrupts work differently in real vs protected mode), [[os-boot-sequence]] (IDT is set up during early init)

**Sources**: `raw/Expanded Main Page - OSDev Wiki.md`

**Last updated**: 2026-04-18

#os

---

## What Are Interrupts?

Interrupts are signals that pause the current execution, save state, and jump to a handler. There are three kinds:

| Kind | Source | Example |
|---|---|---|
| **Hardware interrupt (IRQ)** | External hardware | Timer tick, keyboard press, disk DMA done |
| **Software interrupt** | `int N` instruction | Legacy syscall mechanism (`int 0x80`) |
| **Exception** | CPU fault/trap | Page fault, divide-by-zero, GPF |

---

## Interrupt Descriptor Table (IDT)

The IDT is a table of 256 entries. Each entry (gate descriptor) tells the CPU: "when interrupt N fires, jump to this handler at this privilege level."

Gate types:
- **Interrupt gate**: clears IF (disables further interrupts) on entry.
- **Trap gate**: does not clear IF. Used for exceptions where re-entrancy is acceptable.
- **Task gate**: rarely used, switches TSS.

Load the IDT with the `lidt` instruction pointing to an `IDTR` struct (base address + limit).

### x86-64 IDT Entry Layout (simplified)

```
[offset low 16] [selector] [IST] [type/attr] [offset mid 16] [offset high 32] [reserved]
```

Key fields: the handler offset (split across three fields), the GDT code segment selector, and the type/attribute byte (gate type + DPL).

---

## Exception Vectors 0–31

The first 32 IDT entries are reserved for CPU exceptions:

| Vector | Name | Cause |
|---|---|---|
| 0 | Divide Error (#DE) | `div` or `idiv` by zero |
| 6 | Invalid Opcode (#UD) | Unknown instruction |
| 8 | Double Fault (#DF) | Exception while handling exception |
| 13 | General Protection Fault (#GP) | Segment violation, bad privilege |
| 14 | Page Fault (#PF) | Missing/protected page — `CR2` has the faulting address |
| 16 | x87 FP Exception | Floating-point error |

**Page fault (14)** is the most important — your handler must check `CR2` (faulting address) and the error code to decide whether to map a page, grow the stack, or kill the process.

**Double fault (8)**: fires when an exception occurs while handling another exception. Always set up a double fault handler with its own known-good stack (via IST in the TSS) or a triple fault (instant reboot) will occur.

---

## IRQ Lines and the PIC

Hardware devices signal the CPU via **IRQ lines** (Interrupt Request). Classic x86 has two **8259 PICs** (Programmable Interrupt Controllers) chained together, providing 15 usable IRQ lines.

| IRQ | Device |
|---|---|
| 0 | PIT (timer) |
| 1 | PS/2 Keyboard |
| 3–4 | Serial ports |
| 8 | RTC |
| 12 | PS/2 Mouse |
| 14–15 | ATA hard disks |

The PIC maps IRQs to IDT vectors. By default, IRQ0–7 map to vectors 8–15, which **conflict** with CPU exceptions. You must remap the PIC (typically to vectors 32–47) immediately during init.

After handling an IRQ, you must send an **End Of Interrupt (EOI)** signal (`outb(0x20, 0x20)`) or the PIC won't send more IRQs.

---

## APIC (Advanced PIC)

Modern systems use the **APIC** instead of the legacy 8259 PIC. There are two parts:

- **Local APIC (LAPIC)**: one per CPU core. Receives IRQs delivered to that core. Also provides the **APIC timer** (per-core timer, much more flexible than the PIT).
- **I/O APIC**: sits on the bus, receives IRQs from hardware, routes them to the appropriate LAPIC.

To use the APIC:
1. Detect it via CPUID.
2. Map the LAPIC registers (MMIO, default at `0xFEE00000`).
3. Parse the **ACPI MADT table** (see [[os-hardware]]) to find the I/O APIC address and IRQ routing.
4. Disable the legacy 8259 PIC (`outb(0xA1, 0xFF); outb(0x21, 0xFF)`).
5. Program the I/O APIC redirection table entries.
6. Send EOI to LAPIC (`lapic[0xB0] = 0`) after each interrupt.

---

## Non-Maskable Interrupt (NMI)

The NMI (vector 2) cannot be masked by clearing `IF`. It signals serious hardware errors (memory parity errors, watchdog timeouts). In OS dev, you mostly set up a handler that logs what it can and halts — there's rarely a recovery path.

---

## The Timer Interrupt

The PIT (or APIC timer) fires at a configured frequency (e.g., 100 Hz = every 10ms). This is the heartbeat of preemptive [[os-scheduling|scheduling]]:

```
Timer IRQ fires every 10ms
  → ISR runs
  → Decrement current process's remaining quantum
  → If quantum expired: call scheduler → context switch
  → EOI
  → iretq
```

Configuring the PIT: write a divisor to port `0x40`. The PIT runs at ~1.193182 MHz, so `divisor = 1193182 / hz`.

---

## Interrupt Service Routines (ISRs)

An ISR is the handler code for one interrupt vector. Key rules:

1. **Save all registers** the interrupted code might be using (or use an ISR stub that does this).
2. **Send EOI before returning** (for hardware IRQs).
3. **Keep ISRs short**. Do minimal work — set a flag, queue data, wake a thread. Do real work outside the IRQ context.
4. **Don't call blocking functions** in an ISR. If you hold a spinlock and try to sleep, you deadlock.

Typical pattern: ISR acknowledges the device, copies data to a buffer, and wakes a kernel thread that does the real processing.

---

## Syscall Interface

System calls are how user programs request kernel services. Two mechanisms on x86-64:

- **Legacy `int 0x80`**: triggers a software interrupt, handled via IDT. Slow (full interrupt overhead).
- **`syscall`/`sysret` instructions**: modern fast path. Uses MSRs (`STAR`, `LSTAR`, `SFMASK`) to define the kernel entry point and CS/SS selectors. Much faster — no IDT lookup.

The kernel entry for `syscall` must be set in `IA32_LSTAR` MSR. Arguments pass in registers (`rdi`, `rsi`, `rdx`, `r10`, `r8`, `r9`); return value in `rax`.
