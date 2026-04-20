# OS Development Overview

**Summary**: A roadmap and topic map for building an operating system from scratch, covering kernel design, hardware interaction, memory management, and the full toolchain.

**Sources**: `raw/Expanded Main Page - OSDev Wiki.md`

**Last updated**: 2026-04-18

#os

---

## What is OS Dev?

Operating system development means writing software that runs directly on hardware with no underlying OS beneath it. You manage memory yourself, talk to hardware via I/O ports and memory-mapped registers, and build every abstraction — processes, files, the scheduler — from scratch.

The OSDev community maintains a wiki with 756+ articles covering every aspect of this. This knowledge cluster extracts and organizes those topics.

---

## Recommended Learning Order

1. [[os-toolchain]] — Set up your cross-compiler, assembler, and emulator (QEMU) first. You cannot build without these.
2. [[os-x86-modes]] — Understand how the CPU starts up in Real Mode and how you move to Protected Mode / Long Mode.
3. [[os-boot-sequence]] — Learn the boot process: BIOS/UEFI → bootloader (GRUB/Limine) → your kernel entry point, GDT.
4. [[os-interrupts]] — Wire up the IDT, ISRs, PIC/APIC. Without interrupts, your kernel is a dead loop.
5. [[os-kernel-models]] — Decide: monolithic, microkernel, exokernel, or modular? This shapes everything downstream.
6. [[os-memory-management]] — Implement paging, a page frame allocator, and a heap allocator.
7. [[os-scheduling]] — Add processes and threads; implement context switching and a scheduling algorithm.
8. [[os-ipc]] — Let processes communicate: shared memory, message passing, synchronization primitives.
9. [[os-hardware]] — Drive real hardware: storage (ATA/NVMe), video, input, networking.

---

## Topic Map

| Topic | One-liner |
|---|---|
| [[os-toolchain]] | GCC cross-compiler, NASM, LD, QEMU/Bochs — the build environment |
| [[os-x86-modes]] | Real Mode → Protected Mode → x86-64 long mode |
| [[os-boot-sequence]] | BIOS/UEFI → GRUB/Limine → kernel entry, GDT setup |
| [[os-interrupts]] | IDT, ISRs, exceptions, PIC, APIC, NMI |
| [[os-kernel-models]] | Monolithic vs microkernel vs exokernel vs modular |
| [[os-memory-management]] | Segmentation, paging, page frame allocation, heap, MMU |
| [[os-scheduling]] | Processes, threads, context switching, scheduling algorithms |
| [[os-ipc]] | Message passing, shared memory, RPC, semaphores, mutexes |
| [[os-hardware]] | CPU, storage, video, I/O, PCI/USB, UEFI/ACPI |

---

## Key Decision Points

**Language choice**: C is traditional. C++ is common. Rust is increasingly viable for new projects (see [[rust-overview]]). Assembly is unavoidable for early boot and context switching regardless of your main language.

**Kernel model** (see [[os-kernel-models]]): Monolithic is simpler and faster; microkernel is more robust but harder. Pick monolithic first unless you have a reason not to.

**Bootloader**: Use Limine or GRUB rather than rolling your own. They handle the painful early boot work and give you a clean environment to start from.

**Testing**: Use QEMU for fast iteration. Bochs is slower but has a better debugger. Real hardware last.
