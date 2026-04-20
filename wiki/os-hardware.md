# OS Hardware Interaction

**Summary**: How an OS talks to hardware — CPU I/O ports and MMIO, storage controllers, video output, input devices, PCI/USB bus enumeration, and ACPI/UEFI firmware tables.

**Pre-Req**: [[os-interrupts]] (most devices signal the CPU via IRQs), [[os-boot-sequence]] (ACPI tables are located via the bootloader)

**Sources**: `raw/Expanded Main Page - OSDev Wiki.md`

**Last updated**: 2026-04-18

#os

---

## How Kernels Talk to Hardware

Two mechanisms:

**I/O Ports**: x86 has a separate 16-bit I/O address space (0x0000–0xFFFF). Access via `in`/`out` instructions. Legacy devices (PIC, PIT, PS/2, serial) live here.

```c
static inline void outb(uint16_t port, uint8_t val) {
    asm volatile("outb %0, %1" : : "a"(val), "Nd"(port));
}
static inline uint8_t inb(uint16_t port) {
    uint8_t val;
    asm volatile("inb %1, %0" : "=a"(val) : "Nd"(port));
    return val;
}
```

**Memory-Mapped I/O (MMIO)**: Device registers appear at specific physical addresses. The kernel maps them into virtual address space via the page table and accesses them with normal load/store instructions. Modern devices (LAPIC, PCI devices, GPU framebuffer) use MMIO.

---

## CPU

### Instruction Set Architecture (ISA)

The ISA defines what instructions the CPU can execute. x86 is CISC (Complex Instruction Set Computer) — many instructions, variable length, complex addressing modes. ARM is RISC — fewer, fixed-length instructions, explicit load/store model.

For OS dev you primarily target x86-64. ARM is relevant for embedded and mobile (Raspberry Pi bare-bones is an entry point — see [[os-boot-sequence]]).

### CPU Registers (see [[os-x86-modes]])

### Model-Specific Registers (MSRs)

MSRs are per-CPU registers for configuration not covered by standard registers. Access via `rdmsr`/`wrmsr` instructions with the MSR number in `ecx`.

Key MSRs:

| MSR | Number | Purpose |
|---|---|---|
| `IA32_EFER` | 0xC0000080 | Long mode enable (bit 8), NX enable (bit 11) |
| `IA32_STAR` | 0xC0000081 | Syscall CS/SS selectors |
| `IA32_LSTAR` | 0xC0000082 | Syscall entry point (64-bit) |
| `IA32_GS_BASE` | 0xC0000101 | GS base (for per-CPU data) |
| `IA32_KERNEL_GS_BASE` | 0xC0000102 | Kernel GS base (swapped on syscall) |

---

## Timers

### PIT (Programmable Interval Timer)
The classic x86 timer chip. Runs at ~1.193 MHz. Configure via ports 0x40–0x43. Fires IRQ 0 (see [[os-interrupts]]). Simple but coarse — the go-to for getting a timer tick working first.

```c
void pit_set_frequency(uint32_t hz) {
    uint32_t divisor = 1193182 / hz;
    outb(0x43, 0x36);                    // channel 0, lobyte/hibyte, square wave
    outb(0x40, divisor & 0xFF);          // low byte
    outb(0x40, (divisor >> 8) & 0xFF);  // high byte
}
```

### APIC Timer
Per-CPU timer built into the Local APIC. Much more flexible than the PIT — you can configure it per-core, which is essential for SMP scheduling. Requires calibrating against a known time source (PIT or HPET) to determine its frequency.

### HPET (High Precision Event Timer)
MMIO-based, very accurate (≥ 10 MHz). Good for one-shot timers and calibrating the APIC timer. Located via the ACPI HPET table.

### CMOS/RTC (Real-Time Clock)
Keeps wall-clock time when the system is off. Access via ports 0x70/0x71. Read at boot to get the current time; don't poll for ticks.

---

## Storage

### ATA / PATA
The classic IDE interface. Still emulated by QEMU. Access via I/O ports (primary: 0x1F0–0x1F7, 0x3F6). Simple to implement for a first disk driver.

PIO mode: CPU reads/writes data one sector at a time. Slow but straightforward. DMA mode: disk writes directly to RAM via ISA DMA, CPU is free.

### SATA / AHCI
Modern serial ATA. Uses the AHCI (Advanced Host Controller Interface) — a PCI device. Much faster than ATA. Access its registers via MMIO (found via PCI enumeration).

AHCI uses command lists and FIS (Frame Information Structure) for communication. More complex than ATA but necessary for real hardware.

### NVMe
Storage over PCIe. Very high throughput, low latency. An NVMe controller is a PCI device with MMIO registers. Commands go through submission/completion queue pairs. Required for modern SSDs.

---

## Video

### VGA Text Mode
80×25 character grid. The framebuffer is at physical address `0xB8000`. Each character is 2 bytes: ASCII code + attribute byte (foreground/background color). Easiest possible display output — works in Real Mode and Protected Mode.

```c
volatile uint16_t *vga = (uint16_t*)0xB8000;
vga[0] = (uint16_t)('H' | (0x07 << 8)); // white 'H' on black
```

### Linear Framebuffer
Modern bootloaders (GRUB via Multiboot2, Limine) give you a pixel framebuffer. You get a base address, resolution (width × height), and pixel format (e.g. BGR8). Drawing a pixel is `framebuffer[y * pitch + x * bpp] = color`.

This lets you render anything — text (via a bitmap font), graphics, a full GUI.

### GPU
Driving a real GPU (Intel, AMD, NVIDIA) requires parsing the GPU's own MMIO register spec or using a display protocol like DisplayPort. Very complex; beyond early OS dev. Use the framebuffer the bootloader gives you.

---

## Input

### PS/2 Keyboard
The classic keyboard interface. Data comes via IRQ 1, read from port 0x60. The byte you receive is a **scancode** — not an ASCII character. You must translate via a scancode table. Two bytes for key press, same with bit 7 set for key release.

### PS/2 Mouse
IRQ 12, same controller (port 0x64 for commands, 0x60 for data). Requires initializing the mouse via the PS/2 controller before it will send data.

### USB HID
Modern keyboards/mice are USB HID devices. Requires a full USB stack (XHCI or EHCI driver). Significantly more complex — tackle PS/2 first.

---

## PCI / PCIe

PCI is the bus that connects most modern devices (AHCI controller, NVMe, GPU, network card). **PCI enumeration** finds all connected devices.

Each PCI device is identified by a (Bus, Device, Function) triple. Accessing configuration space:

```c
uint32_t pci_read(uint8_t bus, uint8_t dev, uint8_t func, uint8_t offset) {
    uint32_t addr = (1u << 31) | (bus << 16) | (dev << 11) | (func << 8) | (offset & 0xFC);
    outl(0xCF8, addr);
    return inl(0xCFC);
}
```

Key config space fields:
- `0x00`: Vendor ID + Device ID — used to identify what the device is.
- `0x08`: Class code — e.g. 0x01 = mass storage, 0x02 = network, 0x03 = display.
- `0x10–0x24`: BARs (Base Address Registers) — where the device's MMIO regions are.

PCIe uses MMIO-based configuration space (ECAM) instead of I/O ports, mapped via the ACPI MCFG table.

---

## USB

USB is a tree of hubs and devices. To talk to USB devices you need a Host Controller driver:
- **UHCI/OHCI**: old USB 1.x. Rarely targeted.
- **EHCI**: USB 2.0. Still common in QEMU.
- **XHCI**: USB 3.x+, also handles USB 2.0 in compatibility mode. The modern target.

USB is complex — the host controller, device enumeration, USB class drivers (HID, mass storage, CDC) are all separate layers. Don't start here; implement PS/2 or serial first.

---

## ACPI and UEFI Firmware Tables

### ACPI (Advanced Configuration and Power Interface)
ACPI tables describe the hardware configuration to the OS: what devices exist, their interrupt routing, power management states, etc.

Key tables:
- **RSDP**: Root System Description Pointer — the starting point. Bootloaders give you its address.
- **RSDT/XSDT**: Root table, lists all other tables.
- **MADT**: Multiple APIC Description Table — tells you LAPIC addresses, I/O APIC addresses, and IRQ override mappings. Essential for APIC setup (see [[os-interrupts]]).
- **HPET**: High Precision Event Timer location.
- **MCFG**: PCIe ECAM base addresses for MMIO config space.

Parsing ACPI tables is tedious but mostly mechanical: walk from RSDP → XSDT → find the table by signature (4-byte ASCII). Validate the checksum (sum of all bytes mod 256 == 0).

### UEFI Runtime Services
After calling `ExitBootServices()`, UEFI boot services are gone, but **runtime services** remain accessible: you can get/set NVRAM variables, query the time, and do firmware-specific things. Map the runtime services into your kernel's address space if you need them.
