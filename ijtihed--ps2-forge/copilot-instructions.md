## ps2-forge

> A tiny, agentic-first **PlayStation 2 game engine** in C. One header, ~20

# ps2-forge - agent guide

A tiny, agentic-first **PlayStation 2 game engine** in C. One header, ~20
core functions (plus a canvas-style sugar layer). This file is the full
contract: read it and you can author, build,
and run a PS2 game without reading the engine source.

## Mental model

A game is **one C file** that fills a `Scene` (three callbacks) and calls
`app_run`. The engine owns the loop, the GS (graphics), the pad, and timing.

```c
#include "engine.h"

typedef struct { int x, y; } Game;          /* your state */

static void init  (void *s, Ctx *c){ }      /* once, before the loop */
static void update(void *s, Ctx *c){ }       /* once per frame: read input, mutate state */
static void render(void *s, Ctx *c){ }       /* once per frame: draw */

int main(void){
    static Game g;
    Scene sc = { .state=&g, .init=init, .update=update, .render=render };
    Config cfg = config_default();           /* cfg.clear_r/g/b = background */
    app_run(cfg, &sc);                        /* never returns */
}
```

Coordinate space is **virtual 320x240** (top-left origin, +y down); the engine
scales it to the GS framebuffer. Loop runs at the console's vsync (~60Hz NTSC).

## API (everything you can call)

Drawing (call only from `render`):
- `e_rect(c, x,y,w,h, r,g,b)` - filled rectangle (0-255 colour).
- `e_text(c, x,y, r,g,b, "STR")` - bitmap text (A-Z 0-9 . - ! ? : *). Uppercased.
- `e_quad(c, x0,y0, x1,y1, x2,y2, x3,y3, r,g,b)` - filled quad (rotated shapes/beams).
- `Sprite` + `e_sprite_init(&spr, rgba, w,h)` then `e_sprite_draw(c,&spr, x,y,w,h)`
  - textured sprite from a CT32 RGBA buffer (transparent texels should be black).
- `e_sprite_draw_tinted(c,&spr, x,y,w,h, r,g,b)` - modulate a white sprite to a colour.
- `Image` + `e_image_draw(c,&img, rgba, iw,ih, dx,dy,dw,dh)` - blit a w*h RGBA
  buffer scaled to a rect, re-uploaded each frame. Use for grids / framebuffers
  (cellular automata, software renderers). Buffer must be 16-byte aligned.
- `e_scissor(c, x,y,w,h)` / `e_scissor_off(c)` - hardware clip rectangle.

3D (built-in software voxel renderer; coords centered on origin, ~-16..+16):
- `e3d_begin(c, yaw, pitch)` - clear the 3D buffer, set the rotation.
- `e3d_voxel(x,y,z, r,g,b)` - queue a depth-shaded voxel (call many times).
- `e3d_end(c)` - depth-sort + blit fullscreen (one draw). Then draw 2D HUD on top.
  Example (a spinning cube of points):
  ```c
  static float yaw;
  void render(void *s, Ctx *c){
      yaw += 0.02f;
      e3d_begin(c, yaw, 0.4f);
      for (int z=-6; z<=6; z++) for (int y=-6; y<=6; y++) for (int x=-6; x<=6; x++)
          if (x==-6||x==6||y==-6||y==6||z==-6||z==6)   /* shell */
              e3d_voxel(x,y,z, 80+x*12, 80+y*12, 200);
      e3d_end(c);
      e_text(c, 6,6, 255,255,255, "3D");
  }
  ```

Input (call from `update`; edge-detected helpers compare to last frame):
- `ctx_is_held(c, BTN_x)` - held this frame.
- `ctx_just_pressed(c, BTN_x)` / `ctx_just_released(c, BTN_x)`.
- Buttons: `BTN_UP BTN_DOWN BTN_LEFT BTN_RIGHT BTN_CROSS BTN_CIRCLE BTN_SQUARE
  BTN_TRIANGLE BTN_START BTN_SELECT BTN_L1 BTN_R1 BTN_L2 BTN_R2 BTN_L3 BTN_R3`.
- `c->frame` - frame counter (uint).

Audio (optional; SFX = embedded ADPCM):
- `e_audio_init(audsrv_irx, irx_size)` - once in `init` (needs an embedded audsrv.irx).
- `int h = e_sfx_load(adpcm_bytes, size)` ; `e_sfx_play(h)`.

Config: `Config{ u8 clear_r, clear_g, clear_b; }`, get a default with `config_default()`.

## Porting a JS/TS game

If you're bringing over an HTML5-canvas or p5.js game: include `engine/canvas.h`
for `cv_*` sugar (`cv_fill`/`cv_rect`/`cv_text`/`cv_circle`/`cv_key`/`cv_image`)
that maps near 1:1 to canvas calls. Split the `requestAnimationFrame` loop into
`update`+`render` (call `cv_begin(c)` at the top of each), scale coords to
320x240, embed assets as C arrays. Full table + worked example in `PORTING.md`;
the `port-js-to-ps2` skill automates it.

## Conventions for agents

- All gameplay math in the 320x240 virtual space. Keep it integer where you can.
- `static` your game state and any large arrays (they live in `.bss`; the EE has 32MB).
- No `malloc` needed for small games. No floats required, but `-lm` is linked.
- One file per game. Copy `examples/template/` and edit `game.c` + the Makefile's `EE_BIN`.
- Assets: bake images/audio into C arrays (a `pack.py` that emits `assets.h`); the
  ELF boots with no filesystem, so embed everything.

## Toolchain (one-time)

```sh
tools/bootstrap.sh          # downloads the prebuilt ps2dev toolchain (no sudo)
# then export the PS2DEV / PS2SDK / GSKIT + PATH it prints
```

## Build

With the env set (above):
```sh
make            # produces <name>.elf
```
The Makefile links: `-lgskit_toolkit -lgskit -ldmakit -laudsrv -lpad -lm -lc`.

## Test loop (the agentic part)

```sh
make test       # build -> boot headless -> prints "RENDER: PASS|FAIL" + exit code
```
`make test` boots the ELF in Play! under Xvfb, screenshots it, and checks the
game area actually drew varied, non-black pixels. So the loop is **edit -> one
command -> verdict** (no eyeballing). `make shot` runs the identical check and
also writes `shot.png`; use whichever target name reads better to you. Deps for
these: Play! in PATH, Xvfb, mesa/llvmpipe,
python3 + mss + Pillow.

## Run

- **Emulator (no BIOS):** Play! - `Play --elf game.elf`. Bind keys in its
  controller config (arrows + a key for Cross). Headless capture: run under
  Xvfb + software GL and screenshot.
- **Real PS2:** copy the `.elf` to USB / memory card, launch via FMCB / wLaunchELF.

## Gotchas (learned the hard way)

- Don't override `gsGlobal->Mode/Width/Height/Field` after `gsKit_init_global()`;
  it desyncs the display and blanks the screen. The engine handles screen setup.
- Textured sprites: this gsKit path treats modulate `0xFF` as full brightness
  (not `0x80`). The engine uses `0xFF`; if you call gsKit directly, match it.
- For text over arbitrary backgrounds the engine uses an alpha-tested font atlas;
  one textured quad per glyph (fast). Don't draw text with per-pixel rects.
- Grid/CA rendering: write colours into one RGBA buffer and `e_image_draw` it,
  rather than thousands of `e_rect` calls.

---
> Source: [Ijtihed/ps2-forge](https://github.com/Ijtihed/ps2-forge) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-28 -->
