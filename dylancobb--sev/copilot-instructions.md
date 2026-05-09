## sev

> This file provides guidance to LLMs when working with code in this repository.

# AGENTS.md

This file provides guidance to LLMs when working with code in this repository.

## Project Overview

Emacs-like extensible text editor built with SDL3, Clay (immediate-mode UI), and Chibi Scheme for scripting/configuration. Targets desktop (Linux/Mac/Windows) and WebAssembly (browser via Emscripten).

## Build Commands

```bash
# Desktop build (CMake)
cmake -S . -B build-desktop
cmake --build build-desktop

# WebAssembly build
emcmake cmake -S . -B build-wasm
cmake --build build-wasm

# Quick build with Makefile (GCC, includes debug options)
make compile          # Build and run
make build-debug      # With AddressSanitizer
make vg               # Run with Valgrind

# Run
./out/sev             # Makefile build
./build-desktop/app   # CMake build
```

## Architecture

### SDL3 Callback-Based Lifecycle

The app uses SDL3's main callbacks pattern (no `main()` function):

- `SDL_AppInit()` in `src/init.c` — window, fonts, Clay UI, buffers, Scheme init
- `SDL_AppIterate()` in `src/iterate.c` — main loop with 60 FPS cap for animations
- `SDL_AppEvent()` in `src/event.c` — event dispatch (keyboard, mouse, window)
- `SDL_AppQuit()` in `src/quit.c` — cleanup

### Global State (`src/state.h`)

`AppState` struct holds all application state, passed through SDL3 callbacks via `void *appstate`. Key sub-structs:

- `Chibi` — Scheme context (`ctx`), global environment (`env`), cached `call_interactively`
- `UIState` — current theme symbol, role/palette `VarTable`s, pre-interned `CachedRoles`, global `scale_factor`
- `InputState` — active keymaps, last key event, mouse click/drag callbacks, `FocusTarget` (PANE / WELCOME / MINIBUFFER), which-key intercept state
- `Minibuf` — active buffer, prompt, submit/cancel Scheme callbacks, push/pop frame stack (depth 8)
- `WhichKeyState` — active flag, enabled flag, intercept keymap, accumulated prefix string
- Macro recording fields — `macro_recording`, `macro_buf`, `macro_buf_len`, `macro_target_reg`

### Text Model (`src/text/`)

Buffers, marks, logical lines, registers, undo, jump list, buffer-local variables.

- See `src/text/README.md` for subsystem details.

### Input + Command Layer (`src/command/`)

Input handling, command execution, mode registry, macro recording, Scheme interpreter init.

- See `src/command/README.md` for subsystem details.

### Scheme Integration (`src/command/scheme.c`, `scheme/`)

Chibi Scheme provides extensibility and user scripting.

- See `scheme/README.md` for subsystem details.

### UI Rendering (`src/clay/`, `src/display/`)

Clay immediate-mode UI with SDL3 renderer backend. Layout defined in `layout.c`:

- Global header (app icon strip)
- Pane tree: binary tree of splits and `PANE_DISPLAY` leaves; each leaf owns its own tab list and renders its own tab bar above the buffer view
- Welcome screen when no panes are open
- Global status bar (mode icon + mode name pill; macro indicator dot + "REC" when recording)
- Which-key popup overlay when active
- Bottom strip: minibuffer (prompt + editable text + cursor) when active, echo area otherwise

- See `src/display/README.md` for display subsystem details.
- See `src/clay/README.md` for Clay library and renderer backend details.

## Key Patterns

- **Dirty flag**: `state->needs_redraw` controls when to re-render
- **Key parsing**: `parse_key_sequence()` handles Emacs-style notation (`"C-x"`, `"M-f"`, `"SPC"`, `"RET"`)
- **Color resolution**: `ui_resolve_color(state, role_symbol)` — two-level role → palette → `Clay_Color`
- **Buffer-local vars from C**: `vartable_get(buffer_get_locals(buf), sexp_intern(ctx, "name", -1), default)`
- **Scheme global vars from C**: `sexp_env_ref(ctx, state->chibi.env, sym, default)`

## Testing

- The Unity testing library (by throwtheswitch.org) is included in `/test/unity/`

- See `test/README.md` for info about testing and common flows to add tests.

## Dependencies

Vendored as git submodules in `vendored/`:

- SDL3 — windowing and input
- SDL_ttf — font rendering
- chibi-scheme — Scheme interpreter

Clone with `--recursive` or run `git submodule update --init --recursive`.

## Plan Mode

- Make the plan extremely concise. Sacrifice grammar for the sake of concision.
- At the end of each plan, give me a list of unresolved questions to answer, if any.

---
> Source: [dylancobb/sev](https://github.com/dylancobb/sev) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-03 -->
