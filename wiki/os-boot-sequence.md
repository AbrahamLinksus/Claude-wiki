# OS Boot Sequence

**Summary**: How a computer goes from power-on to your kernel running — the BIOS/UEFI handoff, bootloader, GDT setup, and early kernel initialization.

**Pre-Req**: [[os-x86-modes]] (boot starts in Real Mode), [[os-toolchain]] (you need a cross-compiler and GRUB/Limine before any of this works)

**Sources**: `raw/Expanded Main Page - OSDev Wiki.md`

**Last updated**: 2026-04-18

#os

---

## The Boot Chain

```
Power on
  → CPU resets, starts executing at 0xFFFFFFF0 (reset vector)
  → BIOS POST (Power-On Self Test)
  → BIOS loads first 512 bytes of boot disk into 0x7C00 (MBR)
  → Bootloader (GRUB/Limine) runs
  → Bootloader loads your kernel ELF into memory
  → Bootloader jumps to kernel entry point
  → Your kernel runs
```

With UEFI the early steps differ (see below), but the end result is the same: your kernel gets control.

---

## BIOS Boot

Legacy BIOS is a 16-bit firmware that runs in Real Mode (see [[os-x86-modes]]). It:

1. Runs POST — checks RAM, discovers devices.
2. Finds a bootable device (checks MBR signature `0x55 0xAA` at bytes 510–511).
3. Loads the 512-byte MBR to `0x7C00` and jumps to it.

From the MBR, a bootloader like GRUB takes over. GRUB's Stage 1 is 512 bytes; Stage 2 is larger and handles loading your kernel.

---

## UEFI Boot

UEFI is a 32/64-bit firmware with a full mini-OS. It:

1. Reads a GPT-partitioned disk, finds the **EFI System Partition** (FAT32 formatted).
2. Loads and executes a `.efi` application (your bootloader or kernel directly).
3. Provides boot services (memory map, file I/O, GOP graphics) until you call `ExitBootServices()`.

UEFI is more complex to target directly. Using Limine (which supports UEFI) lets you avoid writing a UEFI application yourself.

---

## Bootloaders

### GRUB (Grand Unified Bootloader)
The most common Linux bootloader. Supports Multiboot and Multiboot2 protocols — your kernel declares a multiboot header and GRUB handles loading it, passing you a `multiboot_info` struct with the memory map, framebuffer info, and module list.

### Limine
Modern bootloader with a clean protocol. Supports both BIOS and UEFI. Highly recommended for new OS projects — the protocol is simpler than Multiboot2 and actively maintained.

Limine gives your kernel:
- A memory map (usable RAM ranges)
- A framebuffer (pixel buffer address, dimensions)
- RSDP pointer (for ACPI — see [[os-hardware]])
- Kernel virtual/physical address

### Rolling Your Own
Possible but painful. You're writing 16-bit assembly, dealing with the A20 line, loading sectors from disk, setting up protected mode — all before your kernel even starts. Use GRUB or Limine unless the bootloader itself is the learning goal.

---

## Global Descriptor Table (GDT)

The GDT is a table in memory that defines memory segments for the CPU. Even in 64-bit mode where segmentation is mostly irrelevant, a valid GDT is required.

Minimum GDT for a 64-bit kernel:

| Entry | Type | Description |
|---|---|---|
| 0 | Null descriptor | Required by spec — always null |
| 1 | Code segment | Kernel code (64-bit, ring 0) |
| 2 | Data segment | Kernel data (ring 0) |
| 3 | Code segment | User code (64-bit, ring 3) |
| 4 | Data segment | User data (ring 3) |
| 5 | TSS | Task State Segment (required for syscalls/interrupts) |

Load the GDT with the `lgdt` instruction, then reload segment registers (`cs`, `ds`, `ss`, etc.).

---

## Task State Segment (TSS)

The TSS is a special GDT entry the CPU uses during privilege level changes (ring 3 → ring 0 on interrupts/syscalls). Its most important field: `rsp0` — the kernel stack pointer the CPU loads when an interrupt arrives while in user mode. Without a TSS, interrupts while in user mode will use the wrong stack and crash.

---

## Early Kernel Init Checklist

After the bootloader jumps to your kernel entry:

1. Set up a stack (bootloader may give you one, or set `rsp` manually).
2. Load your GDT.
3. Set up the IDT (see [[os-interrupts]]).
4. Parse the memory map from the bootloader.
5. Initialize the page frame allocator (see [[os-memory-management]]).
6. Enable paging / set up kernel page tables.
7. Set up the heap allocator.
8. Enable interrupts (`sti`).
9. Start the scheduler.

---

## The A20 Line

A legacy quirk: on original PCs, address line 20 was gated (always 0), limiting RAM to 1 MB. Some BIOSes leave A20 disabled. If you're not using a modern bootloader that enables it for you, you must enable A20 manually (via port `0x92` or the keyboard controller) or you'll silently get aliased memory accesses above 1 MB.

Modern bootloaders (GRUB, Limine) handle this for you.
