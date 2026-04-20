# OS Development Toolchain

**Summary**: The build environment for OS development — the GCC cross-compiler, assemblers (NASM/GAS), the linker (LD), and emulators (QEMU, Bochs) for testing.

**Pre-Req**: Basic familiarity with compiling C programs

**Sources**: `raw/Expanded Main Page - OSDev Wiki.md`

**Last updated**: 2026-04-18

#os

---

## Why a Cross-Compiler?

A cross-compiler runs on your host OS but produces binaries for a different target — in this case, bare-metal x86-64 with no OS underneath. Using your system GCC is a common beginner mistake: it links against your host's libc and assumes a host OS exists, producing binaries that will crash or silently misbehave when run as a kernel.

**You must build a GCC cross-compiler targeting `x86_64-elf`** (or your target architecture).

---

## GCC Cross-Compiler

### Target: `x86_64-elf`

The triple `x86_64-elf` means: x86-64 architecture, no vendor, bare-metal ELF output (no OS ABI). This gives you a compiler that produces standalone binaries with no host OS assumptions.

### Building the cross-compiler

High-level steps:
1. Download binutils and GCC source.
2. Build **binutils** first (`as`, `ld`, `objcopy`, etc.) configured for `--target=x86_64-elf`.
3. Build **GCC** configured for `--target=x86_64-elf --enable-languages=c,c++ --without-headers`.
4. Install to a prefix (e.g., `$HOME/opt/cross`), add to `$PATH`.

OSDev Wiki maintains a detailed tutorial and a list of known-working version combinations (see `raw/Expanded Main Page - OSDev Wiki.md` → Compilers section).

### Key compiler flags for kernel code

```makefile
CFLAGS = -ffreestanding   # no standard library assumed
         -fno-stack-protector  # no __stack_chk_fail dependency
         -fno-pic          # no position-independent code (unless needed)
         -mno-red-zone     # x86-64: disable red zone (breaks with interrupts)
         -nostdlib         # don't link standard libraries
         -O2
```

`-mno-red-zone` is critical: the x86-64 ABI's 128-byte "red zone" below `rsp` is unusable in kernel code because interrupts can clobber it.

---

## Assemblers

### NASM (Netwide Assembler)
Intel syntax, widely used in OS dev. Clean, explicit, well-documented. Most OSDev tutorials use NASM.

```nasm
section .text
global _start
_start:
    mov rax, 60    ; syscall: exit
    xor rdi, rdi   ; status 0
    syscall
```

### GAS (GNU Assembler, `as`)
AT&T syntax by default (src and dst operands swapped vs Intel). Can be used with `.intel_syntax noprefix` to get Intel syntax. Part of binutils — always available when you have GCC.

### FASM, YASM
Alternatives. FASM is self-hosting (written in assembly). YASM is largely NASM-compatible. Either works; NASM has the most tutorials.

### AT&T vs Intel Syntax

| | Intel | AT&T |
|---|---|---|
| Operand order | `dst, src` | `src, dst` |
| Register prefix | none | `%` prefix (e.g. `%rax`) |
| Immediate prefix | none | `$` prefix (e.g. `$5`) |
| Memory | `[rsp+8]` | `8(%rsp)` |
| Size suffix | `dword ptr` | `l`/`q` suffix on mnemonic |

---

## Linker (LD)

The linker combines compiled object files into your final kernel binary. For OS dev you write a **linker script** that precisely controls the memory layout of your kernel.

Minimal linker script for a higher-half kernel:

```ld
ENTRY(_start)
SECTIONS {
    . = 0xFFFFFFFF80000000;   /* kernel virtual base (higher half) */

    .text ALIGN(4K) : AT(ADDR(.text) - KERNEL_VMA) {
        *(.text*)
    }
    .rodata ALIGN(4K) : {
        *(.rodata*)
    }
    .data ALIGN(4K) : {
        *(.data*)
    }
    .bss ALIGN(4K) : {
        *(COMMON)
        *(.bss*)
    }
}
```

The linker script controls:
- Where sections land in virtual memory (`.` = location counter).
- `AT(addr)` — the physical load address (where it actually goes in RAM).
- Section alignment (page-align everything for paging to work cleanly).

---

## Emulators

### QEMU
The standard tool for OS dev testing. Fast, scriptable, excellent debug support.

```bash
qemu-system-x86_64 \
  -kernel mykernel.elf \
  -serial stdio \       # serial output to terminal
  -no-reboot \          # don't reboot on triple fault (you want to see it)
  -d int,cpu_reset      # log interrupts and CPU resets
```

Use `-s -S` to start a GDB server: `qemu-system-x86_64 -s -S ...`, then in GDB: `target remote :1234`.

### Bochs
Slower than QEMU but has a built-in debugger with excellent x86 internals visibility — you can step through instructions, inspect registers, view the GDT/IDT, check page tables. Useful when QEMU's output doesn't tell you enough.

### VMware / VirtualBox
Useful for testing hardware compatibility, but less convenient for kernel debugging than QEMU/Bochs. Use after your kernel is more mature.

### Real Hardware — Last
Boot on real hardware last, after the emulator works. Differences: UEFI quirks, ACPI tables vary by board, hardware that emulators don't emulate (NVMe timing, USB controllers, etc.).

---

## Working with Disk Images

Your kernel needs to be on a bootable disk image for GRUB/Limine to load it.

```bash
# Create a 64 MB disk image
dd if=/dev/zero of=disk.img bs=1M count=64

# Partition it (GPT + EFI System Partition for UEFI)
parted disk.img mklabel gpt
parted disk.img mkpart ESP fat32 1MiB 33MiB
parted disk.img set 1 esp on

# Mount and install Limine + kernel
losetup --find --show --partscan disk.img  # → /dev/loopX
mkfs.fat -F32 /dev/loopXp1
mount /dev/loopXp1 /mnt
cp kernel.elf /mnt/
limine bios-install disk.img
umount /mnt && losetup -d /dev/loopX
```

On Linux, the **loopback device** (`losetup`) lets you mount a disk image as if it were a real disk.

---

## Debugging Tips

- Always compile with `-g` (debug info) even for kernel code.
- Use `objdump -d kernel.elf` to disassemble and find where crashes happen.
- QEMU's `-d int` logs every interrupt — invaluable for triple faults.
- GDB + QEMU remote (`-s -S`) lets you set breakpoints, inspect registers, step through kernel code.
- Print to serial port (port `0x3F8`) early — you have serial output before you have a VGA driver.
