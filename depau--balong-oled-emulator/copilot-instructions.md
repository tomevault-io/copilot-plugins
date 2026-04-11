## balong-oled-emulator

> This project contains both an emulator for the LCD/OLED display of some Huawei Balong-based portable routers and a

# Balong OLED emulator and custom menu

This project contains both an emulator for the LCD/OLED display of some Huawei Balong-based portable routers and a
library to add a custom menu to the OLED display of Huawei E5785 portable LTE router.

The custom menu libraries work by hijacking certain library calls in the "oled" executable file with `LD_PRELOAD`.

The emulator provides those functions for the library to hook, and displays the rendered framebuffer using SDL2.

Both the emulator and the custom menu support models with a 128x128 16-bit BGR565 LCD displays as well as models with
128x64 1-bit b/w monochrome OLED displays.

The custom menu library uses Clay with a custom renderer to render the menus. Currently only a "hello world" menu is
implemented, but it will be extended in the future.

## Project structure

- `emulator/` - The Balong OLED/LCD emulator using SDL2.
- `custom-menu/` - The custom menu library.
- `include/` - Common headers
- `generate_bitmap_font.py` - Script that converts a given TTF font to a bitmap font to embed in the custom menu
  library.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/depau)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/depau)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
