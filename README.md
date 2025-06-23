# Lab 1

## Part 1: PC Bootstrap

### The PC's Physical Address Space

A PC's physical address space is hard-wired to have the following layout:

```
+------------------+  <- 0xFFFFFFFF (4GB)
|      32-bit      |
|  memory mapped   |
|     devices      |
|                  |
/\/\/\/\/\/\/\/\/\/\

/\/\/\/\/\/\/\/\/\/\
|                  |
|      Unused      |
|                  |
+------------------+  <- depends on amount of RAM
|                  |
|                  |
| Extended Memory  |
|                  |
|                  |
+------------------+  <- 0x00100000 (1MB)
|     BIOS ROM     |
+------------------+  <- 0x000F0000 (960KB)
|  16-bit devices, |
|  expansion ROMs  |
+------------------+  <- 0x000C0000 (768KB)
|   VGA Display    |
+------------------+  <- 0x000A0000 (640KB)
|                  |
|    Low Memory    |
|                  |
+------------------+  <- 0x00000000
```

### The ROM BIOS

On power-up or after system restart, Intel processors always enter real mode and set CS=0xf000 and IP=0xfff0.

This is done because the BIOS is "hard-wired" to the physical address range 0x000f0000-0x000fffff, and Intel wants to ensure that the BIOS always gets control of the machine first.

#### Exercise 2 - what is the BIOS doing?

The BIOS sets up an interrupt vector table, and inits various devices.
After initing every all the known important devices, the BIOS searches for a bootable
device (e.g: hard-drive) and reads the bootloader from it.

## Part 2: The Boot Loader

Hard-drives / floppy discs are divided into 512 byte blocks called *sectors* - which are the disk's minimum transfer granularity.

If the disk is bootable, the first sector is called the boot sector - and the boot loader code resides there. 

When the BIOS finds a bootable floppy or hard disk, it loads the 512-byte boot sector into memory at physical addresses 0x7c00 through 0x7dff, and then uses a jmp instruction to set the CS:IP to 0000:7c00, passing control to the boot loader.

The bootloader first switches to 32-bit protected mode, and set up segment translation that makes virtual addresses identical to their physical addresses.

Then, it reads the kernel ELF from the disc, using each segment's specified physical as its actual physical address.

What's interesting is the while the bootloader loads the kernel at a low address, if we look at the link address for the kernel we'll see a very high one - which can cause some concerns... 

## Part 3: The Kernel

### Using virtual memory to work around position dependence

Operating system kernels often like to be linked and run at very high virtual address, such as 0xf0100000.

Many machines don't have any physical memory at address 0xf0100000, so we can't count on being able to store the kernel there. 

Eventually, we'll set up "normal" virtual mappings for the kernel.

However, for now, we'll just map 0xf0000000 through 0xf0400000 to physical addresses 0x00000000 through 0x00400000, as well as virtual addresses 0x00000000 through 0x00400000 to physical addresses 0x00000000 through 0x00400000.

### Formatted Printing to the Console

**Exercise 8:** See lib/printfmt.c:vprintfmt to see my implementation of "%o".

### The Stack

After enabling paging, the kernel sets up a temporary stack which it allocates
statically at the data section, calls the i386_init function which initializes the console devices, and tests our work.