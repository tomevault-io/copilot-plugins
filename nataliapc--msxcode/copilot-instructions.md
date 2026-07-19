## msxcode

> **msxcode** is an MSX 2/2+/tR application written in C (SDCC compiler) that acts as an AI assistant client

# AGENTS.md — MSXcode Project Guide

## Project Overview

**msxcode** is an MSX 2/2+/tR application written in C (SDCC compiler) that acts as an AI assistant client
running on real MSX hardware. It communicates with AI backends over TCP/IP UNAPI and displays a TUI
(Text User Interface) using a custom ANSI widget library.

- **Target hardware**: MSX 2/2+/TurboR (Z80 compatible)
- **OS**: MSX-DOS 2.x or higher
- **Compiler**: SDCC 4.2.0 via Docker (`nataliapc/sdcc:4.2.0`)
- **Output**: `dsk/msxcode.com` (MSX-DOS executable)
- **Screen**: V9938 / V9958 VDP, Screen 7 (G7, 512×212, 16 colors), 85×26 text columns via msx2ansi

---

## Directory Structure

```
tool_msxcode/
├── src/                    Main application source files (C/ASM)
│   ├── msxcode.c           Entry point, main loop
│   ├── mod_disposable.c    Init, screen setup, printHeader() — DISPOSABLE segment
│   ├── mod_commandLine.c   Command-line argument parsing
│   ├── mod_help.c          Help screen
│   ├── heap.c              Heap allocator
│   ├── crt0msx_msxdos_advanced.s  Custom CRT0 for MSX-DOS
│   └── libs/               Library source files
│       ├── acmp_*.c        ANSI TUI widget implementations
│       ├── vdp_*.c/.s      VDP helper routines
│       └── utils_*.c       Utility functions
├── includes/               Header files
│   ├── ansi_components.h   TUI widget API declarations
│   ├── ansi_codes.h        ANSI/VT52/VT100 escape code macros
│   ├── msx2ansi.h          msx2ansi library API
│   ├── msx_const.h         MSX system constants and variables
│   └── ...
├── contrib/                Third-party libraries (source + prebuilt)
│   └── msx2ansi/           ANSI rendering library for V9938
│       ├── src/
│       │   ├── msx2ansi.asm        Main ANSI engine (Z80 ASM)
│       │   ├── msx2ansibuffer.asm  Buffer helper
│       │   └── msx2ansi.sh         Build script (runs inside Docker)
│       └── msx2ansi.lib    Built library (copied to libs/ by make)
├── externals/              Git submodules
│   ├── sdcc_msxconio/      conio (console I/O) library
│   └── sdcc_msxdos/        MSX-DOS 2 API library
├── libs/                   Compiled libraries used by the linker
│   ├── msx2ansi.lib        ANSI rendering engine
│   ├── ansi_components.lib TUI widgets
│   ├── vdp.lib             VDP helpers
│   ├── utils.lib           Utility functions
│   ├── conio.lib           Console I/O
│   └── dos.lib             MSX-DOS 2 API
├── dsk/                    Disk image folder (mounted in openMSX)
│   └── msxcode.com         Final compiled executable
├── plan/                   Implementation plans (PRPs)
│   ├── PRP001_acmp_label.md
│   ├── PRP002_acmp_panel.md
│   ├── PRP003_acmp_progressBar.md
│   ├── PRP004_acmp_lines.md
│   ├── PRP005_acmp_badge.md
│   ├── PRP006_acmp_menu.md
│   ├── PRP007_acmp_confirm.md
│   ├── PRP008_acmp_inputBox.md
│   ├── PRP009_acmp_textArea.md
│   └── PRP010_hpost.md
├── obj/                    Compiler output (.rel, .ihx, .map, .com)
├── res/                    Resources (logo, help text — compressed with zx0)
├── emulation/              openMSX boot scripts
└── Makefile                Main build system
```

---

## Build System

All compilation uses SDCC 4.2.0 via Docker (`nataliapc/sdcc:4.2.0`).
`hex2bin` must be installed locally to convert `.ihx` → `.com`.

### Build the full project (most common)

```sh
make
```

Compiles everything: contrib libs → externals → src libs → application.
Output: `dsk/msxcode.com`

### Rebuild only the application (after changing src/*.c)

```sh
make
```

The Makefile detects changed files automatically via dependencies.

### Choosing the right clean target (IMPORTANT — avoid unnecessary cleans)

Use the minimal clean needed to avoid wasting compilation time. Combine targets when multiple
areas changed. **Never use `make clean` unless everything is broken.**

| What changed | Clean command before `make` |
|---|---|
| Only `src/*.c` / `src/*.s` (main program files) | `make cleanprogram` |
| Only `src/libs/*.c` or `includes/*.h` (project libs) | `make cleanprogram` |
| Only `res/` (resources / help text) | `make cleanres` |
| Only `contrib/msx2ansi/src/msx2ansi.asm` | `make cleancontrib` |
| Only `contrib/UNAPI_TCPIP/src/*.c` | `make cleancontrib` |
| Any other contrib lib | `make cleancontrib` |
| `externals/sdcc_msxconio` or `externals/sdcc_msxdos` | `make cleanlibs` |
| Mix of contrib + project libs | `make cleancontrib cleanprogram` |
| Mix of resources + program | `make cleanres cleanprogram` |
| Everything broken / first build | `make clean` |

**`cleanprogram`** removes only the compiled `.rel` files for the main program and project libs
(listed in `REL_LIBS`), forcing them to relink without touching extern/contrib artifacts.

**`cleancontrib`** removes only contrib build artifacts (`contrib/` subtree + `libs/` copies of
contrib libs such as `msx2ansi.lib`, `unapi_tcpip.lib`). Always required after editing any file
under `contrib/`.

**`cleanlibs`** removes all libs including externals (`conio.lib`, `dos.lib`) plus everything
`cleancontrib` removes. Only needed when externals change.

**`cleanres`** removes compiled resource `.c` files and `utils.lib` (which embeds the resources).
Required after editing anything in `res/`.

```sh
# Examples
make cleanprogram && make          # changed src/msxcode.c
make cleancontrib && make          # changed contrib/msx2ansi/src/msx2ansi.asm
make cleanres cleanprogram && make # changed res/ AND src/libs/utils_help_zx0.c
make cleanlibs && make             # changed externals (rare)
make clean && make                 # nuclear option — only when truly needed
```

> **Note on contrib**: `make contrib` alone will NOT recompile if the `.lib` already exists.
> Always pair with `cleancontrib` after editing any `contrib/` source.

---

## Testing in openMSX

The emulator is configured as: `turbor` machine + disk folder `dsk/` + the extensions
`debugdevice` and `unapinet`.

> **IMPORTANT — always launch openMSX with BOTH extensions:**
> ```json
> "extensions": [
>     "debugdevice",
>     "unapinet"
> ]
> ```
> `unapinet` is mandatory: without it the app fails at boot with *"TCP/IP UNAPI not found"*
> (the AUTOEXEC reports `openMSX UnapiNet extension not found. Load it with: ext unapinet`).
> `debugdevice` is required for the debug workflow. When launching via the `mcp-openmsx`
> tools, pass `extensions: ["debugdevice", "unapinet"]` to `emu_control.launch`.

### Launch from Makefile

```sh
make test
```

### Manual launch (if emulator already running)

Press **ESC** in the app to return to MSX-DOS, then type:

```
msxcode
```

No reset needed — the `dsk/` folder is mounted live.

---

## Key Technical Notes

### msx2ansi Color Model (CRITICAL)

The msx2ansi library maps ANSI SGR codes to V9938 palette indices:

| SGR range | Effect |
|-----------|--------|
| `30-37`   | ForeColor = index 0-7 |
| `40-47`   | BackColor = index 0-7 |
| `90-97`   | ForeColor = index 0-7 + HiLighted=1 (adds 8 at render time) |
| `100-107` | BackColor = index 0-7 + HiLightBG=1 (adds 8 at render time) — **implemented in this project** |
| `1` (bold)| HiLighted=1 → FG index +8 (bright foreground) |
| `0`       | Reset all: ForeColor=7, BackColor=0, HiLighted=0, HiLightBG=0 |

- `HiLightBG` was NOT implemented in the original msx2ansi v1.7 — it was added here.
- `csprintf` in SDCC does NOT support `%3d`, `%-5s` or any width/padding specifiers.
  Build numbers manually using arithmetic.
- LSP errors from clangd are ALL false positives — clangd does not understand SDCC headers.

### Screen Layout

- **85 columns × 26 rows** of 6×8 pixel characters
- Cursor hardware disabled globally (`ESC x 5`) in `initializeScreen()`
- Only `acmp_inputBox` (PRP008) re-enables the cursor temporarily

### Widget Library (ansi_components)

Widgets implemented (see `src/libs/acmp_*.c`, declared in `includes/ansi_components.h`):

| Widget | Function | Notes |
|--------|----------|-------|
| `acmp_label` | Text with L/C/R alignment | Static, no buffer needed |
| `acmp_panel` | Box with border + optional title | PANEL_SIMPLE / PANEL_DOUBLE |
| `acmp_progressBar` | Progress bar with optional % | PCT_NONE / PCT_AFTER / PCT_INSIDE |
| `acmp_hLine` / `acmp_vLine` | Horizontal/vertical lines | Various line chars |
| `acmp_badge` / `acmp_badgeFull` | Colored badge labels | Returns total width |
| `acmp_messageBox` | Simple message box | Uses panel + label |

Pending: `acmp_menu` (PRP006), `acmp_confirm` (PRP007), `acmp_inputBox` (PRP008).

### Color Strings in Widget API

Colors are passed as SGR parameter strings (without `\033[` and `m`):

```c
acmp_label(1, 1, 20, LABEL_LEFT, "0;37;42", "Hello");  // white on green
acmp_label(1, 2, 20, LABEL_LEFT, "1;33",    "Bold yellow");
acmp_label(1, 3, 20, LABEL_LEFT, NULL,      "Default color");
```

Always start with `0;` to reset previous attributes, then add color codes.

### PCT_INSIDE progressBar rendering rule

- Filled zone: print `' '` (space) with barColor BG — so the BG color is visible
- Empty zone: print `░` (`\xB0`) with emptyColor — gives texture via fg/bg mix
- `pctColor` = SGR for the `%` number text drawn over the filled zone

### SAVESCREEN / RESTORESCREEN

Modal widgets use:
```c
AnsiSaveScreen();  // ESC[?47h
// ... render modal ...
AnsiRestoreScreen();  // ESC[?47l
```

---

## Debugging in openMSX

Use the `mcp-openmsx` MCP tools. Key workflow for MSX-DOS programs:

1. **Never set breakpoints before the app is on screen** — BIOS/DOS boot will fire them first.
2. Always check `selectedSlots` before reading RAM (all pages should be slot 3.0 = RAM).
3. Always check `isBreaked` before trying to read memory or continue.
4. The msx2ansi lib is loaded as part of `_CODE` segment starting at `0x07C0`.
5. Symbol addresses can be found in `obj/msxcode.map` for C functions, but msx2ansi
   internal symbols (BackColor, ColorTable, etc.) are local and not exported to the map.
   Find them by disassembling and pattern-matching known instruction sequences.
6. Use a `getch()` before the test code to wait for a keypress and give yourself time to open the debugger and set breakpoints, then send a `\r` key.
7. When reading memory check before the slots and the emu status (is it breaked?) to avoid false data.

---

## Engram Persistent Memory — Protocol

You have access to Engram, a persistent memory system that survives across sessions and compactions.

---
> Source: [nataliapc/msxcode](https://github.com/nataliapc/msxcode) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-19 -->
