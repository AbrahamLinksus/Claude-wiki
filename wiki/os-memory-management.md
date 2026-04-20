# OS Memory Management

**Summary**: How an OS controls physical and virtual memory — segmentation, paging, page frame allocation, heap allocation, and the MMU's role.

**Pre-Req**: [[os-x86-modes]], [[os-boot-sequence]] (paging is set up during early kernel init)

**Sources**: `raw/Expanded Main Page - OSDev Wiki.md`

**Last updated**: 2026-04-18

#os

---

## The Problem

Multiple processes need to share physical RAM without stepping on each other. Memory management solves this through hardware-assisted address translation and allocation policies.

---

## Segmentation

Segmentation divides memory into named regions (segments) with a base address and limit. The CPU translates a logical address `(segment, offset)` into a linear address by adding the segment base.

On x86, the **Global Descriptor Table (GDT)** defines segments. See [[os-boot-sequence]] — the GDT is one of the first things you set up.

In modern 64-bit (long mode) OS dev, segmentation is mostly vestigial. Most OSes set up flat segments covering the full address space and rely entirely on paging. You still need a valid GDT, but you don't use segmentation for memory isolation.

---

## Paging

Paging is the dominant memory management mechanism. The CPU's MMU translates **virtual addresses** → **physical addresses** using a page table tree.

### How it works

- Physical memory is split into fixed-size **page frames** (4 KB on x86).
- Virtual memory is split into same-sized **pages**.
- A page table maps virtual page numbers → physical frame numbers.
- On x86-64, this is a 4-level tree: PML4 → PDPT → PD → PT.

```
Virtual address: [PML4 idx | PDPT idx | PD idx | PT idx | offset]
                      ↓           ↓         ↓        ↓
                 PML4 table → PDPT → PD → Page Table → Physical frame
```

### Benefits
- **Isolation**: each process has its own virtual address space; pages not mapped = access fault.
- **Flexibility**: non-contiguous physical frames can appear contiguous in virtual space.
- **Demand paging**: pages can be loaded from disk on first access (page fault handler).

### TLB

The Translation Lookaside Buffer (TLB) caches recent virtual→physical translations. Changing page table entries requires a TLB flush (`invlpg` or `mov cr3`).

---

## Page Frame Allocator

Before you can set up paging, you need to track which physical frames are free. The simplest approach:

1. **Bitmap allocator**: one bit per frame; set = allocated. `O(n)` allocation, very simple.
2. **Stack allocator**: push free frames onto a stack; `O(1)` allocation.
3. **Buddy allocator**: Linux-style; splits and merges power-of-2 blocks; efficient, low fragmentation.

Your bootloader (GRUB/Limine) gives you a memory map via multiboot or via a framebuffer info struct. Parse it to find usable RAM ranges and initialize your allocator from those.

---

## Heap / Dynamic Allocation

The page frame allocator gives out 4 KB chunks. For `malloc`-style fine-grained allocation, you implement a **heap allocator** on top:

- **Bump allocator**: pointer advances on alloc, no free. Useful for initial kernel setup only.
- **Free list allocator**: linked list of free blocks; first-fit or best-fit search.
- **Slab allocator**: pre-allocate caches of fixed-size objects; fast for kernel structs. Used by Linux.

---

## Memory Management Unit (MMU)

The MMU is the hardware unit that performs page table walks. Key x86 registers:

| Register | Purpose |
|---|---|
| `CR0` | Bit 31 (PG) enables paging |
| `CR3` | Physical address of top-level page table (PML4 on x86-64) |
| `CR2` | Faulting virtual address on a page fault |

A **page fault** (#PF exception, vector 14) fires when a page is not present, not writable, or a protection violation occurs. Your page fault handler inspects `CR2` and the error code to decide: map the page, grow the stack, or kill the process.

---

## Virtual Address Space Layout (Typical x86-64)

```
0xFFFF_FFFF_FFFF_FFFF ┐
  Kernel space         │  Kernel mapped here; protected from user mode
0xFFFF_8000_0000_0000 ┘
         ...           (non-canonical hole)
0x0000_7FFF_FFFF_FFFF ┐
  User space           │  Each process gets its own mapping here
0x0000_0000_0000_0000 ┘
```

---

## File Systems

File management is memory management applied to storage. At the VFS layer, files are just byte arrays; the file system (FAT32, ext2, etc.) maps them to disk blocks. See [[os-hardware]] for storage driver details that sit beneath this layer.
