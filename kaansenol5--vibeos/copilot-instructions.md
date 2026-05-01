## vibeos

> **IMPORTANT DISK RULES:**

# VibeOS - Claude Context

**IMPORTANT DISK RULES:**
- Never look at the disk
- The disk always works
- `make run` is the only way to compile and run the code - the user will run it
- Trust `make run` - the user will tell you if it is broken
- Do NOT use hdiutil for anything

## Project Overview
VibeOS is a hobby operating system built from scratch for aarch64 (ARM64), targeting QEMU's virt machine. This is a science experiment to see what an LLM can build.

## The Vibe
- **Aesthetic**: Modern macOS-inspired (draggable windows, dock, menu bar)
- **Philosophy**: Simple, educational, nostalgic
- **NOT trying to be**: Linux, production-ready, or modern

## Current State (Last Updated: Session 40)
- [x] Bootloader (boot/boot.S) - Sets up stack, clears BSS, jumps to kernel
- [x] Minimal kernel (kernel/kernel.c) - UART output working
- [x] Linker script (linker.ld) - Memory layout for QEMU virt
- [x] Makefile - Builds and runs in QEMU
- [x] Boots successfully! Prints to serial console.
- [x] Memory management (kernel/memory.c) - malloc/free working, dynamic heap sizing
- [x] DTB parsing (kernel/dtb.c) - Detects RAM size at runtime from Device Tree
- [x] String functions (kernel/string.c) - memcpy, strlen, strcmp, strtok_r, etc.
- [x] Printf (kernel/printf.c) - %d, %s, %x, %p working
- [x] Framebuffer (kernel/fb.c) - ramfb device, 800x600
- [x] Bitmap font (kernel/font.c) - 8x16 VGA-style font
- [x] Console (kernel/console.c) - Text console with colors on screen
- [x] Virtio keyboard (kernel/keyboard.c) - Full keyboard with shift support
- [x] Shell (kernel/shell.c) - In-kernel shell with commands
- [x] VFS (kernel/vfs.c) - Now backed by FAT32, falls back to in-memory
- [x] Coreutils - ls, cd, pwd, mkdir, touch, rm, cat, echo (with > redirect)
- [x] ELF loader (kernel/elf.c) - Loads PIE binaries with full relocation support
- [x] Process management (kernel/process.c) - Process table, context switching, scheduler
- [x] Cooperative multitasking - yield(), spawn() in kapi, round-robin scheduler
- [x] Kernel API (kernel/kapi.c) - Function pointers for programs to call kernel
- [x] Text editor (kernel/vi.c) - Modal vi clone with normal/insert/command modes
- [x] Virtio block device (kernel/virtio_blk.c) - Read/write disk sectors
- [x] FAT32 filesystem (kernel/fat32.c) - Read/write, full LFN (long filename) support
- [x] Persistent storage - 64MB FAT32 disk image, mountable on macOS
- [x] Interrupts - GIC-400 working! Keyboard via IRQ, boots at EL3 (Secure)
- [x] Timer - 10ms tick (100Hz), used for uptime tracking
- [x] System Monitor - GUI app showing uptime and memory usage
- [x] TextEdit - GUI text editor with Save As modal
- [x] RTC - PL031 real-time clock at 0x09010000, shows actual date/time
- [x] Date command - /bin/date shows current UTC date/time
- [x] Menu bar - Apple menu with About/Quit, File menu, Edit menu
- [x] About dialog - Shows VibeOS version, memory, uptime
- [x] Power management - WFI-based idle, mouse interrupt-driven, 100Hz UI refresh
- [x] Virtio Sound - Audio playback via virtio-sound device, WAV and MP3 support
- [x] Music Player - GUI music player with album/track browser, pause/resume, progress bar
- [x] Floating point - FPU enabled, context switch saves/restores FP regs, calc uses doubles
- [x] Networking - virtio-net driver, Ethernet, ARP, IP, ICMP working!
- [x] Ping command - `/bin/ping` can ping internet hosts (1.1.1.1, etc.)
- [x] UDP + DNS - hostname resolution via QEMU's DNS server (10.0.2.3)
- [x] TCP - full TCP state machine with 3-way handshake, send/recv, close
- [x] HTTP client - `/bin/fetch` can make HTTP requests to real websites!
- [x] Web Browser - `/bin/browser` GUI browser with HTML rendering, works on HTTP sites
- [x] TLS 1.2 - Full HTTPS support via TLSe library, ECC key exchange working!
- [x] HTTPS client - `/bin/fetch` can make HTTPS requests to google.com, etc!
- [x] Raspberry Pi Zero 2W support - boots on real hardware!
- [x] Pi SD card (EMMC) driver - FAT32 filesystem works, userspace runs!
- [x] Pi USB host (DWC2) - Device enumeration working, HID keyboard via hub works!

## Architecture Decisions Made
1. **Target**: QEMU virt machine, aarch64, Cortex-A72
2. **Memory start**: 0x40000000 (QEMU virt default)
3. **UART**: PL011 at 0x09000000 (QEMU virt default)
4. **Stack**: 64KB, placed in .stack section after BSS
5. **Toolchain**: aarch64-elf-gcc (brew install)
6. **Compiler flags**: -mstrict-align (prevent unaligned SIMD), FPU enabled
7. **Process model**: Win3.1 style - no memory protection, programs run in kernel space

## Roadmap (Terminal-First)
Phase 1: Kernel Foundations - DONE
1. ~~Memory management~~ - heap allocator working
2. ~~libc basics~~ - string functions, sprintf
3. ~~Display~~ - framebuffer, console, font
4. ~~Keyboard~~ - virtio-input with shift keys
5. ~~Shell~~ - in-kernel with basic commands
6. ~~Filesystem~~ - in-memory VFS

Phase 2: Programs - MONOLITH APPROACH
7. ~~ELF loader~~ - working but abandoned
8. ~~Kernel API~~ - kapi struct with function pointers
9. **DECISION**: Monolith kernel - all commands built into shell
   - Tried external programs, hit linker issues with 6+ embedded binaries
   - Win3.1 vibes - everything in one binary is fine

Phase 3: Apps (DONE)
10. ~~Text editor~~ - vi clone with modal editing (normal/insert/command modes)
11. ~~Snake~~ - moved to /bin/snake userspace program
12. ~~Tetris~~ - moved to /bin/tetris userspace program

Phase 4: GUI (IN PROGRESS)
13. ~~Mouse driver~~ - virtio-tablet support
14. ~~Window manager~~ - /bin/desktop with draggable windows, close boxes
15. ~~Double buffering~~ - reduces flicker
16. ~~Terminal emulator~~ - /bin/term with stdio hooks
17. ~~Visual refresh~~ - True 1-bit B&W System 7 aesthetic
18. Notepad/text editor, file explorer - TODO
19. DOOM?

## Technical Notes

### QEMU virt Machine Memory Map
- 0x00000000 - 0x3FFFFFFF: Flash, peripherals
- 0x08000000: GIC (interrupt controller)
- 0x09000000: UART (PL011)
- 0x0A000000: RTC
- 0x0A003E00: Virtio keyboard (device 31)
- 0x40000000+: RAM (we load here)
- 0x41000000+: Program load area (dynamically allocated by kernel)

### Key Files
- boot/boot.S - Entry point, EL3→EL1 transition, BSS clear, .data copy
- kernel/kernel.c - Main kernel code
- kernel/dtb.c/.h - Device Tree Blob parser (RAM detection)
- kernel/memory.c/.h - Heap allocator (malloc/free), dynamic sizing
- kernel/string.c/.h - String/memory functions
- kernel/printf.c/.h - Printf implementation
- kernel/fb.c/.h - Framebuffer driver (ramfb)
- kernel/console.c/.h - Text console (with UART fallback)
- kernel/font.c/.h - Bitmap font
- kernel/keyboard.c/.h - Virtio keyboard driver (interrupt-driven)
- kernel/irq.c/.h - GIC-400 interrupt controller driver
- kernel/vectors.S - Exception vector table
- kernel/virtio_blk.c/.h - Virtio block device driver
- kernel/fat32.c/.h - FAT32 filesystem driver (read/write)
- kernel/shell.c/.h - In-kernel shell with all commands
- kernel/vfs.c/.h - Virtual filesystem (backed by FAT32 or in-memory)
- kernel/vi.c/.h - Modal text editor (vi clone)
- kernel/elf.c/.h - ELF64 loader (supports PIE binaries)
- kernel/process.c/.h - Process table, scheduler, context switching
- kernel/context.S - Assembly context switch routine
- kernel/kapi.c/.h - Kernel API for programs
- kernel/initramfs.c/.h - Binary embedding (currently unused)
- kernel/virtio_sound.c/.h - Virtio sound driver (WAV playback)
- linker.ld - Memory layout (flash + RAM regions)
- Makefile - Build system
- disk.img - FAT32 disk image (created by `make disk`)

### User Directory (userspace programs)
- user/lib/vibe.h - Userspace library header
- user/lib/gfx.h - Shared graphics primitives (header-only)
- user/lib/icons.h - Dock icons and VibeOS logo bitmaps
- user/lib/crt0.S - C runtime startup
- user/bin/*.c - Single-file program sources
- user/bin/browser/ - Multi-file browser (str.h, url.h, http.h, html.h, main.c)
- user/linker.ld - Program linker script (PIE, base at 0x0)

### Build & Run
```bash
make            # Build kernel AND all user programs (everything)
make clean      # Clean build artifacts
make disk       # Create FAT32 disk image (only needed once)
make run        # Run with GUI window (serial still in terminal)
make run-nographic  # Terminal only
make distclean  # Clean everything including disk image
```

**IMPORTANT**: `make` builds EVERYTHING - kernel and all user programs. There is no separate user-progs target. Just use `make clean && make` to rebuild.

### Mounting the Disk Image (macOS)
```bash
hdiutil attach disk.img        # Mount
# ... add/edit files in /Volumes/VIBEOS/ ...
hdiutil detach /Volumes/VIBEOS # Unmount before running QEMU
```

### Shell Commands
| Command | Description |
|---------|-------------|
| help | Show available commands |
| clear | Clear screen |
| echo [text] | Print text (supports > redirect) |
| version | Show VibeOS version |
| mem | Show memory info |
| pwd | Print working directory |
| ls [path] | List directory contents |
| cd <path> | Change directory |
| mkdir <dir> | Create directory |
| touch <file> | Create empty file |
| rm <file> | Remove file |
| cat <file> | Show file contents |
| vi <file> | Edit file (modal editor) |

### VFS Structure
```
/
├── bin/        (empty - monolith kernel)
├── etc/
│   └── motd    (message of the day)
├── home/
│   └── user/   (default cwd)
└── tmp/
```

## Architecture Decisions (Locked In)
| Component | Decision | Notes |
|-----------|----------|-------|
| Kernel | Monolithic | Everything in kernel space, Win3.1-style |
| Programs | PIE on disk | Loaded dynamically at runtime, kernel picks address |
| Memory | Flat (no MMU) | No virtual memory, shared address space |
| Multitasking | Cooperative | Programs call yield(), round-robin scheduler |
| Filesystem | FAT32 on virtio-blk | Persistent, mountable on host, read/write |
| Shell | POSIX-ish | Familiar syntax, basic redirects |
| RAM | Detected via DTB | Works with 256MB-4GB+, heap sized dynamically |
| Disk | 64MB FAT32 | Persistent storage via virtio-blk |
| Interrupts | GIC-400 | Keyboard & mouse via IRQ, boots at EL3 for full GIC access |
| Power | WFI idle | Scheduler sleeps CPU when no work, wakes on interrupt |

## Gotchas / Lessons Learned
- **aarch64 va_list**: Can't pass va_list to helper functions easily. Inline the va_arg handling.
- **QEMU virt machine**: Uses PL011 UART at 0x09000000, GIC at 0x08000000
- **Virtio legacy vs modern**: QEMU defaults to legacy virtio (version 1). Use `-global virtio-mmio.force-legacy=false` to get modern virtio (version 2).
- **Virtio memory barriers**: ARM needs `dsb sy` barriers around device register access.
- **strncpy hangs**: Our strncpy implementation causes hangs in some cases. Use manual loops instead.
- **Static array memset**: Don't memset large static arrays - they're already zero-initialized.
- **GIC Security Groups**: GIC interrupts require matching security configuration. If running in Secure EL1, use Group 0 interrupts. Group 1 interrupts in Secure state return IRQ 1022 (spurious). Boot with `-bios` and `secure=on` to start at EL3 with full GIC register access.
- **-mgeneral-regs-only**: Use this flag to prevent GCC from using SIMD registers.
- **Stack in BSS**: Boot hangs if stack is in .bss section - it gets zeroed while in use! Put stack in separate .stack section.
- **Embedded binaries**: objcopy binary embedding breaks with 6+ programs. Linker issue. Just use monolith kernel instead.
- **Console without framebuffer**: console_puts/putc fall back to UART if fb_base is NULL.
- **kapi colors**: Must use uint32_t for colors (RGB values like 0x00FF00), not uint8_t.
- **Packed structs on ARM**: Accessing fields in `__attribute__((packed))` structs causes unaligned access faults. Read bytes individually and assemble values manually.
- **FAT32 minimum size**: FAT32 requires at least ~33MB. Use 64MB disk image.
- **Virtio-blk polling**: Save `used->idx` before submitting request, then poll until it changes. Don't use a global `last_used_idx` that persists across requests.
- **Virtio-input device detection**: Both keyboard and tablet are virtio-input. Must check device name contains "Keyboard" specifically, not just starts with "Q".
- **Userspace has no stdint.h**: Use `unsigned long` instead of `uint64_t` in user programs, or define types in vibe.h.
- **PIE on AArch64**: Use `-fPIE` and `-pie` flags. ELF loader processes R_AARCH64_RELATIVE relocations at load time.
- **Context switch**: Only need to save callee-saved registers (x19-x30, sp). Caller-saved regs are already on stack.
- **PIE relocations (FIXED!)**: Use `-O0` for userspace to ensure GCC generates relocations for static pointer initializers. With `-O2`, GCC tries to be clever and compute addresses at runtime, but puts structs in BSS (zeroed) so pointers are NULL. The ELF loader now processes `.rela.dyn` section and fixes up `R_AARCH64_RELATIVE` entries. Normal C code with pointers now works!
- **strtok_r NULL rest**: After the last token, `strtok_r` sets `rest` to NULL (not empty string). Always check `rest && *rest` before dereferencing.
- **Flash/RAM linker split**: When booting via `-bios`, code lives in flash (0x0) but data/BSS must be in RAM (0x40000000). Use separate MEMORY regions in linker script with `AT>` for load addresses. Copy .data from flash to RAM at boot.
- **EL3→EL1 direct**: Can skip EL2 entirely. Set `SCR_EL3` with NS=0 (stay Secure), RW=1 (AArch64), then eret to EL1.
- **WFI in scheduler**: When a process yields and it's the only runnable process, WFI before returning to it. This prevents busy-wait loops from cooking the CPU. The kernel handles idle, not individual apps.
- **Don't double-sleep**: If kernel WFIs on idle, apps shouldn't also sleep_ms() - that causes double delay and sluggish UI. Apps just yield(), kernel handles the rest.
- **FAT32 LFN + GCC -O2**: The LFN entry building code crashes with -O2 optimization. Use -O0 for fat32.c. Symptom: translation fault when writing to valid heap memory. Root cause unknown but likely optimizer generating bad code for the byte-by-byte LFN entry construction.
- **Stack must be above BSS**: As kernel grows, BSS section grows. Stack pointer must be well above BSS end. Originally at 0x40010000, but BSS grew to 0x400290d4 - stack was inside BSS and got zeroed during boot! Moved to 0x40100000 (1MB into RAM).
- **_data_load must be 8-byte aligned**: AArch64 `ldr x3, [x0]` instruction requires 8-byte alignment. If `_data_load` in linker script is not aligned, boot hangs during .data copy loop. Add `. = ALIGN(8);` before `_data_load = .;`.
- **Build with -O0 for safety**: GCC optimization causes subtle bugs in OS code - PIE relocations, LFN construction, possibly virtio drivers. Using -O0 everywhere avoids these issues at cost of larger/slower code.
- **FPU enable**: Set CPACR_EL1.FPEN = 0b11 in boot.S to enable FP/SIMD. Without this, any FP instruction causes an exception.
- **FP context switch**: When FPU is enabled, context_switch must save/restore q0-q31, fpcr, fpsr. The fp_regs array must be 16-byte aligned (stp/ldp q regs require this). Added padding to cpu_context_t to ensure fp_regs is at offset 0x80.
- **-mstrict-align required with FPU**: Without -mgeneral-regs-only, GCC uses SIMD for memcpy/struct copies. Some SIMD loads require aligned addresses. Use -mstrict-align to prevent unaligned SIMD access faults.
- **Kernel stack vs heap collision**: Heap runs from `_bss_end + 0x10000` to `0x41000000`. If kernel stack is inside this range, large allocations (like framebuffer backbuffer) will overwrite the stack. Symptom: local variables corrupted with data like `0x00ffffff` (COLOR_WHITE). Stack was at 0x40100000 (inside heap!). Moved to 0x4F000000 (well above heap and program area).
- **DTB at RAM start**: QEMU places the Device Tree Blob at 0x40000000 (start of RAM). Linker script must start .data/.bss after DTB area (we use 0x40200000, leaving 2MB for DTB).
- **DTB unaligned access**: Reading 32/64-bit values from DTB can cause alignment faults on ARM. Read bytes individually and assemble manually (see `read_be32`/`read_be64` in dtb.c).
- **TTF italic buffer overflow**: When applying styles to glyphs, track original glyph width vs allocated stride separately. If a function calculates "extra space needed" and you pass an already-expanded buffer, it will double the expansion and overflow. The `apply_italic` bug wrote past buffer end, corrupting heap metadata.
- **DWC2 USB MPS for Full Speed**: When enumerating Full Speed USB devices, use MPS=64, not 8! FS devices can send up to 64 bytes per packet. Using MPS=8 causes babble error when byte 9 arrives. Only Low Speed devices use MPS=8.
- **DWC2 USB PHY clock on Pi**: Pi uses UTMI+ PHY at 60MHz. Set HCFG.FSLSPCLKSEL=0 (30/60MHz mode), NOT 1 (48MHz). Wrong setting causes premature frame end and babble errors.
- **DWC2 USB multi-packet IN transfers**: In slave mode, must re-enable channel after each packet received. Controller doesn't automatically continue. Re-enable on ACK and IN_COMPLETE events.
- **DWC2 USB GRXSTSP parsing**: Only read FIFO data when pktsts=2 (IN data) and bcnt>0. Other status values (3=complete, 7=halted) don't have data to read.
- **DWC2 USB split transactions broken**: FS devices behind HS hubs require split transactions. The DWC2 split transaction state machine (start-split → complete-split) had issues causing infinite NYET loops. Workaround: Set `HCFG_FSLSUPP` to force Full-Speed only mode. Hub connects at FS, no splits needed. 12 Mbps is plenty for keyboard/mouse.
- **USB msleep must use kernel timer**: The USB driver's msleep() using ARM generic timer (`cntfrq_el0`) didn't work on Pi. Use kernel's `sleep_ms()` which uses timer tick interrupts. Interrupts must be enabled before USB init.

## Session Log
For detailed session-by-session development history, see [session_log.md](session_log.md).

---
> Source: [kaansenol5/VibeOS](https://github.com/kaansenol5/VibeOS) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-20 -->
