# x86 CPU Modes

**Summary**: The progression of CPU operating modes on x86 — Real Mode, Protected Mode, and Long Mode (x86-64) — what each enables, and how to transition between them.

**Pre-Req**: Basic understanding of what a CPU register is

**Sources**: `raw/Expanded Main Page - OSDev Wiki.md`

**Last updated**: 2026-04-18

#os

---

## Why Modes Exist

x86 evolved over decades while maintaining backward compatibility. Each new mode added capabilities (more memory, privilege levels, paging) without breaking existing software. When your CPU powers on today, it starts in the same mode as a 1981 IBM PC — Real Mode — and you must explicitly transition to 64-bit Long Mode.

---

## Real Mode

**What it is**: The original 8086 mode. 16-bit registers, no memory protection, no paging, segmented addressing.

**Memory**: Max 1 MB addressable. Physical address = `(segment × 16) + offset`. The infamous A20 gate limits you further (see [[os-boot-sequence]]).

**Privilege**: One privilege level — everything runs with full hardware access.

**When you're in it**: At power-on, and when BIOS runs. Your bootloader starts here.

**Key constraint**: No virtual memory, no process isolation, no protected mode features. Any code can write anywhere in RAM.

---

## Protected Mode

**What it is**: 32-bit mode introduced with the 80286/80386. Adds memory protection, privilege rings, and segmentation with access checks.

**Memory**: Up to 4 GB addressable (32-bit address space). Segmentation enforced by the GDT (see [[os-boot-sequence]]).

**Privilege rings**:
```
Ring 0 — Kernel (most privileged)
Ring 1 — (rarely used)
Ring 2 — (rarely used)
Ring 3 — User applications (least privileged)
```

Most OSes only use ring 0 (kernel) and ring 3 (user). Attempting a privileged instruction from ring 3 triggers a General Protection Fault (see [[os-interrupts]]).

**Paging**: Optional but standard. Enabled by setting bit 31 of `CR0`. See [[os-memory-management]].

**How to enter from Real Mode**:
1. Load a GDT with at least a null, code, and data descriptor.
2. Set `PE` bit (bit 0) in `CR0`.
3. Far jump to flush the instruction pipeline and load CS from the GDT.
4. Reload `ds`, `ss`, `es`, etc. with the data segment selector.

```asm
cli
lgdt [gdt_descriptor]
mov eax, cr0
or  eax, 1          ; set PE bit
mov cr0, eax
jmp 0x08:protected_mode_entry  ; far jump, 0x08 = kernel code selector
```

---

## Virtual 8086 Mode (V86)

A sub-mode of Protected Mode that emulates a Real Mode environment for legacy 16-bit software. The kernel sets up a "virtual machine" with a 1 MB address space, trapping privileged instructions. Rarely implemented in modern OS dev.

---

## Long Mode (x86-64)

**What it is**: 64-bit mode, introduced with AMD64 (then adopted by Intel as EM64T/Intel 64). The standard for modern OS dev.

**Memory**: Theoretically 2^64 bytes; current CPUs implement 48-bit virtual addresses (256 TB) with the rest being a non-canonical hole. Physical addresses are 52 bits on most hardware.

**Key additions over Protected Mode**:
- 64-bit general-purpose registers (`rax`, `rbx`, ... + `r8`–`r15`).
- `rip`-relative addressing.
- Segmentation mostly deprecated (all segment bases treated as 0; `fs` and `gs` still usable for TLS).
- 4-level paging required (PML4 → PDPT → PD → PT).

**How to enter from Protected Mode**:
1. Enable PAE (Physical Address Extension): set bit 5 in `CR4`.
2. Load a PML4 page table into `CR3`.
3. Enable Long Mode in the `EFER` MSR (bit 8 = LME).
4. Enable paging (`PG` bit in `CR0`) — this activates Long Mode.
5. Far jump to a 64-bit code segment.

```asm
; In 32-bit protected mode:
mov eax, cr4
or  eax, (1 << 5)   ; PAE
mov cr4, eax

mov eax, pml4_addr
mov cr3, eax

mov ecx, 0xC0000080 ; EFER MSR
rdmsr
or  eax, (1 << 8)   ; LME
wrmsr

mov eax, cr0
or  eax, (1 << 31)  ; PG
mov cr0, eax

jmp 0x08:long_mode_entry  ; far jump to 64-bit segment
```

---

## Comparison Table

| Feature | Real Mode | Protected Mode | Long Mode (x86-64) |
|---|---|---|---|
| Register width | 16-bit | 32-bit | 64-bit |
| Max virtual address space | 1 MB | 4 GB | 256 TB (48-bit) |
| Privilege levels | None | 4 rings | 4 rings |
| Paging | No | Optional | Required |
| Segmentation | Yes (simple) | Yes (enforced) | Mostly disabled |
| Typical use | BIOS, bootloader | Old 32-bit OSes | Modern OS kernels |

---

## CPU Registers (x86-64 Quick Reference)

| Register(s) | Purpose |
|---|---|
| `rax`, `rbx`, `rcx`, `rdx` | General-purpose (extended from 32-bit eax etc.) |
| `rsi`, `rdi` | Source/destination (also arg passing) |
| `rsp` | Stack pointer |
| `rbp` | Base pointer (frame pointer) |
| `r8`–`r15` | Additional general-purpose (new in x86-64) |
| `rip` | Instruction pointer |
| `rflags` | Status flags (carry, zero, sign, interrupt enable...) |
| `cr0` | Control: PE, PG bits |
| `cr2` | Page fault linear address |
| `cr3` | Page table base (PML4 physical address) |
| `cr4` | Extended features: PAE, PSE, etc. |
| `cs`, `ds`, `ss`, `es`, `fs`, `gs` | Segment registers (mostly vestigial in long mode) |

See [[os-interrupts]] for how the CPU uses these registers during interrupt handling.
