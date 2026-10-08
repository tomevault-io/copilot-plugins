## cyd-doom

> Port of GBADoom (doomhack's GBA port of PrBoom) to the CYD: ESP32-WROOM (no

# DOOM for the ESP32-2432S028R "Cheap Yellow Display"

Port of GBADoom (doomhack's GBA port of PrBoom) to the CYD: ESP32-WROOM (no
PSRAM), ILI9341_2-variant 320x240 SPI panel, XPT2046 resistive touch. The
board is normally on **COM5** (CH340).

## Build / flash

```
python -m platformio run              # build
python -m platformio run -t upload    # flash firmware (COM5)
python tools/flashwad.py              # flash doom1_cyd.wad to the wad partition
```

Serial monitor at 115200. Keys over serial: w/a/s/d move+turn, q/e strafe,
f fire, space use, m menu, t automap. BOOT button = menu. Touch zones:
left half = d-pad, right half = use (top) / fire (bottom), corners = menu/map.

## Layout

- `src/doom/` - GBADoom engine (C). Lightly patched: see "Port patches".
- `src/port/` - ESP32 platform layer:
  - `main.cpp` - boot, doom task (24KB static stack, core 1)
  - `display.cpp` - SPI panel driver, 240x160 8bpp -> 320x213 RGB565 scaled push
  - `input.cpp` - XPT2046 bit-bang, serial keys, BOOT button
  - `setup.cpp` - first-boot wizard: orientation/colour + touch calibration
  - `wad_mmap.c` - maps the `wad` flash partition to `doom_iwad`
  - `sram.c` - save games + display config in the `sram` partition
- `include/` - engine headers (patched: m_fixed.h, doom_iwad.h, gba_functions.h)
- `partitions.csv` - app 1MB @0x10000, sram 32KB @0x110000, wad 2.9MB @0x118000
- `tools/wadtrim.py` - trims GbaWadUtil-processed WAD to fit flash
- `tools/flashwad.py` - writes the WAD image at 0x118000
- `tools/GbaWadUtil/` - GbaWadUtil.exe WAD preprocessor (from doomhack/GBADoom;
  upstream clone itself was removed - engine source lives in src/doom)

## WAD pipeline

doom1.wad (shareware 1.9) -> `tools\GbaWadUtil\GbaWadUtil.exe -in doom1.wad
-out doom1_gba.wad` -> `python tools/wadtrim.py trim doom1_gba.wad
doom1_cyd.wad --limit 0x2E8000 --drop-maps E1M6,E1M7,E1M9`.
Trim drops audio lumps/demos/3 maps and stubs HELP1/HELP2/CREDIT/WIMAP0 as
1.5KB all-black patches. Kept maps: E1M1-E1M5 + E1M8 (E1M5 exit jumps to M8,
secret exits act as normal exits - patched in g_game.c).

## Hardware gotchas (each cost real debugging time - do not regress)

- **One SPI device per CS pin.** A second device with the same `spics_io_num`
  steals the GPIO matrix routing; the first device then transmits with the
  panel deselected. All readback/diagnostics are bit-banged instead.
- **Panel ignores MADCTL/COLMOD while asleep.** Send SLPOUT before config.
- **SPI DMA cannot read from flash.** Bounce `.rodata` params through DRAM.
- **XPT2046: DIN must be low while clocking the reply** or the high bits are
  taken as a new command and you read the temperature channel (~204 constant).
- DC=GPIO2, CS=15, SCLK=14, MOSI=13, MISO=12, backlight=21 (verified by probe).
  Touch: CLK=25, CS=33, DIN=32, DOUT=39, IRQ=36 (IRQ unused; pressure z1 used).
- Display config (MADCTL etc.) varies per board clone - the setup wizard
  (hold BOOT at power-on to re-run) finds and persists it; don't hardcode.

## Memory budget

ESP32 heap is fragmented (~110KB + ~63KB regions). Z_Init must run before
display/framebuffer allocations and takes the largest block (~110KB zone).
Single 38.4KB framebuffer (two don't fit); frame push is synchronous.
Statics ~75KB incl. 24KB doom-task stack. Game runs ~28 fps.

## Not done yet

- Sound (CYD speaker on GPIO26/DAC2; i_audio.c is stubbed no-ops)
- Music
- E1M9/E1M6/E1M7 dropped for flash space (4MB total)
- Boot-time DC-pin probe + speed negotiation could be trimmed now that the
  answers are known (GPIO2, 40MHz) but they self-heal board variants - keep.

---
> Source: [HenrysCat/cyd-doom](https://github.com/HenrysCat/cyd-doom) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-07 -->
