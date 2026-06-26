## port-commander

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build & run

```bash
cmake -S . -B build
cmake --build build
./build/apcommander                    # dashboard — click a tile to launch a tool
./build/portcommander                  # standalone: network-port inspector
./build/wificommander                  # standalone: nmcli Wi-Fi / hotspot GUI
./build/adc_gui                        # standalone: 24-bit ADC monitor
./build/psu_gui                        # standalone: dual-channel PSU full GUI
./build/psu_gui_single                 # standalone: single-channel PSU full GUI
./build/psu_gui_toolbar                # standalone: dual-channel PSU compact strip
./build/psu_gui_toolbar_single         # standalone: single-channel PSU compact strip
```

System dependencies (Debian/Ubuntu): `build-essential cmake pkg-config libsdl2-dev libsdl2-ttf-dev lsof procps fonts-dejavu-core`. `wificommander` additionally requires `network-manager` (`nmcli`) at runtime, and `qrencode` to display the join-QR (silently degrades when missing). The ADC/PSU tools talk to a serial device (default `/dev/ttyUSB0`, override via argv[1]) and fall back to a demo mode when the port can't be opened.

There is no test suite, no linter config, and no formatter config in the repo. The compile flags `-Wall -Wextra -Wpedantic` are the only enforced checks for the native targets (`portcommander`, `wificommander`, `apcommander`). The imported `adc_gui` / `psu_gui*` targets were brought in with their upstream warnings still tripping (e.g. `usleep` deprecation under `_POSIX_C_SOURCE=200809L`); CMakeLists.txt suppresses a handful of `-Wno-unused-*` and `-Wno-sign-compare` for those targets only — don't propagate those suppressions to the native code. CI just runs `cmake --build` (`.github/workflows/build.yml`) and uploads all eight binaries as one artifact.

## Architecture

Eight executables and one static library, built from one CMakeLists.txt:

- **`apcompat`** (static lib in `compat/`) is the cross-platform abstraction layer used by the dashboard. It exposes: `compat_app_dir` (wraps `SDL_GetBasePath`), `compat_exe_suffix`, `compat_can_execute`, `compat_path_lookup` (walks `$PATH` for a bare name), `compat_home_dir` (HOME with USERPROFILE fallback on Windows), `compat_default_font` (platform-specific monospace-font search), and the spawn/poll pair `compat_spawn` (accepts an optional argv array) / `compat_proc_poll` / `compat_proc_free`. POSIX path uses `fork` + `execvp` + non-blocking `waitpid`; the Windows path (sketched, uncompiled) uses `CreateProcessA` + `WaitForSingleObject` + `GetExitCodeProcess` and naïvely quotes argv into a cmdline. The POSIX child also closes inherited FDs above stderr before `execvp` so spawned tools don't keep the dashboard's X11 socket / font mmaps alive. **All non-portable code in the dashboard goes through this layer** — adding Windows means filling in the `_WIN32` branches there, not touching the dashboard's source files.
- **`apcommander`** (in `dash/`) is the user-facing entry point: an SDL2 dashboard rendered as a curved vertical menu — a fixed selector at the window's vertical center, options scrolling past it, each option's x bending outward along a circular arc as it moves away from the selector (port of smoothui's `fixed_selector_v_stack_curved_menu` example, no dep on raylib/smoothui). The dashboard is split into focused translation units, all linked into one executable: `dash/main.c` is the top-level orchestrator (SDL setup, font loading, main loop, header/footer chrome); `dash/tile.{c,h}` defines the `Tile` struct and its ownership/lifecycle (`tile_free`), shared by every other layer; `dash/config.{c,h}` owns INI parsing, args tokenisation, exec-path resolution, and the config search chain (`config_load`); `dash/menu.{c,h}` owns the curved-menu state, event handling (`menu_handle_event` returns a `MenuAction`), spring animation, layout, and render; `dash/spawn_pool.{c,h}` exposes the opaque `SpawnPool` type — a non-blocking child-process tracker with built-in launch debounce, per-frame reap, and detach-on-exit; `dash/render.{c,h}` provides shared SDL2 drawing primitives (`render_fill` / `render_outline` / `render_text*`) and the dark-theme `COL_*` palette. Tiles are loaded from a user-editable INI config, so adding a new launcher entry doesn't require recompiling. Search order: `argv[1]` → `$APCOMMANDER_CONFIG` → `$XDG_CONFIG_HOME/apcommander/tiles.conf` (or `~/.config/...`) → `<app_dir>/apcommander.conf` → `<app_dir>/apcommander.default.conf` (shipped defaults, copied next to the binary at build time by `CMakeLists.txt`'s `add_custom_command(POST_BUILD …)`). The dashboard never carries tile data baked into the binary — if no config is found anywhere it fails with a list of every path it tried. Each `[section]` is one tile; supported keys are `title`, `desc`, `exec`, `args`, `hotkey`. `exec` accepts absolute paths, `~/`-paths, app-dir-relative paths, or bare names (which are looked up first next to the dashboard binary, then in `$PATH` via `compat_path_lookup`). See `dash/apcommander.default.conf` for the full schema and copy-to-customize instructions. The dashboard stays visible — you can launch multiple tools in parallel (`MAX_CHILDREN=64`), and a green border + `● N running` badge tracks live children per tile. Every frame the loop calls `pool_reap` which polls each tracked child and reaps the exited ones. On exit, `pool_destroy_detach` frees the tracking handles without signalling the children, so they outlive the dashboard. If a tool binary is missing, its tile is shown with a "missing" tag; missing-state is re-checked on launch attempt to close the TOCTOU window. `MAX_TILES=64`; window default 720×660. A 350 ms per-tile launch debounce lives in `pool_launch` and combines with `e.key.repeat == 0` filtering in `menu_handle_event` to swallow OS key auto-repeat and stray double-clicks after a launched window steals focus.
- **`portcommander`** (in `src/`) and **`wificommander`** (in `wifi/`) are the *native* tools: written for this project, Linux-only (`lsof`/`/proc`, `nmcli`), share the architectural shape described below, and are the parts of the codebase you should actively edit.
- **`adc_gui`** (in `adc/`) and the four **`psu_gui*`** binaries (in `psu/`) are *imported* tools: they came in verbatim from the sibling projects `dev-psu-gui` and `dev-modbus/psu-gui`. Treat them as third-party — keep them buildable and runnable, but don't refactor them into the native architecture unless asked. Each has its own copy of `serial_port.{c,h}`; do not deduplicate without explicit instruction (the upstreams may diverge). POSIX-only via `<termios.h>`; a Windows port would need a `serial_port_win32.c` swap.
- **`dmm_gui`** (in `dmm/`) and **`eload_gui`** (in `eload/`) are *native* instrument-style tools, designed to match the imported PSU/ADC look (same color palette, VFD-style readouts, header / mode-bar / footer skeleton). Each is one self-contained `main.c` with no protocol module — they accept a serial device on argv[1] for parity with the other instruments but currently run only in demo mode (the simulator lives at the top of each file). When a real protocol is wired up, follow the imported tools' shape: split into `main.c` + `<tool>_protocol.{c,h}` + `serial_port.{c,h}`.
- **`common/vfd.h`** is a header-only 5×7 dot-matrix VFD glyph renderer extracted from `psu/main.c` (so the imported PSU code stays untouched but the new native tools can match its look). Static-inline functions, no library — each tool that includes it gets its own copy and remains standalone-buildable. Use `vfd_draw_number()` for the readout and `vfd_measure()` to right-align it. Glyph set is `0-9 . - <space>`; non-numeric labels still render through TTF.

### Platform gating in CMake

Three CMake-time platform classes:

| class            | tools                                                                | gate                                          |
| ---------------- | -------------------------------------------------------------------- | --------------------------------------------- |
| Linux-only       | `portcommander`, `wificommander`                                     | `if(CMAKE_SYSTEM_NAME STREQUAL "Linux")`      |
| POSIX            | `adc_gui`, `psu_gui`, `psu_gui_single`, `psu_gui_toolbar`, `psu_gui_toolbar_single` | `if(NOT WIN32)`              |
| cross-platform   | `apcommander`, `apcompat`                                            | always                                        |

`add_dependencies(apcommander …)` at the bottom of CMakeLists.txt only adds whichever targets exist for the host — building `apcommander` produces the dashboard plus every tool the platform supports.

### Standalone capability

Every tool has its own `main()` and SDL window and is built as an independent executable. The dashboard is purely a launcher — there is no shared library between tools, no shared SDL context, no IPC. Building `--target portcommander` produces a single self-contained binary you can ship without `apcommander`. The split layout (one tool per directory, each with its own protocol/UI/main triple) preserves this: do not introduce shared state between tools without an explicit reason.

### Native tool architecture (`portcommander`, `wificommander`)

Both share the same shape — a parsing/data layer, a query layer, an SDL `ui_render` layer, and a `main.c` orchestrator that owns the event loop and is the only place that performs side effects (signals, fork+exec).

### Port Commander (`src/`, target `portcommander`)

Reads sockets from `lsof` / `/proc` and renders a live table.

- **`src/lsof_parse.{c,h}`** — runs `lsof -i -P -n -F` via `popen` and parses its `-F` field-stream output into a `PortTable` of `PortRow{pid, comm, proto, name, state}`.
- **`src/ps_query.{c,h}`** — fills a `ProcessDetail` for a single PID by reading `/proc/<pid>/status`, `/proc/<pid>/cmdline`, and `ps -p <pid> -o etime=`. Called only when the user selects a row.
- **`src/ui_render.{c,h}`** — SDL rendering, scrolling, filter input, two-step kill confirm. Owns the `Ui` struct.
- **`src/main.c`** — event loop, owns the `PortTable` and filtered `visible[]` slice, performs the `kill(pid, SIGTERM)`.

### Wi-Fi Commander (`wifi/`, target `wificommander`)

Wraps `nmcli` for Wi-Fi listing and hotspot start/stop. Built specifically because GNOME's "Turn On Wi-Fi Hotspot" toggle silently fails on some drivers while `nmcli device wifi hotspot` works.

- **`wifi/nmcli_run.{c,h}`** — shared `fork`/`pipe`/`execvp` helpers: `nmcli_spawn_stdout` (read-only queries, FILE\* of stdout), `nmcli_run_capture_stderr` (one-shot actions, captures stderr for error reporting). All `nmcli` invocations go through these so user-supplied SSID/password values are passed as argv elements, never through a shell.
- **`wifi/nmcli_query.{c,h}`** — `wifi_table_refresh` runs `nmcli -t -f IN-USE,SIGNAL,SECURITY,SSID,BSSID device wifi list`. `wifi_state_refresh` discovers the wifi interface, the active wireless connection, and (if `802-11-wireless.mode == ap`) the hotspot SSID/PSK/band. Includes a `split_terse` parser that handles nmcli `-t` mode's `\:` and `\\` escapes (BSSIDs always contain colons).
- **`wifi/nmcli_action.{c,h}`** — `nmcli_hotspot_up` builds the `nmcli device wifi hotspot ifname … ssid … password … [band …]` argv; `nmcli_hotspot_down` runs `nmcli connection down <name>`. Validates password length ≥ 8 and rejects values starting with `-` before forking (defense against argument-injection regressions in nmcli's option parser).
- **`wifi/qr_codec.{c,h}`** — builds a `WIFI:T:WPA;S:<ssid>;P:<password>;;` payload (with the standard `\` escaping for `;:,\\"`), shells out to `qrencode -t UTF8`, and renders the captured Unicode block-character output as filled SDL rectangles. We invert qrencode's white-on-black-terminal convention: `' '` → both modules black, `▀` → bottom black, `▄` → top black, `█` → no-op. Used only when a hotspot is active; falls back to a "install qrencode" hint if the binary is missing.
- **`wifi/ui_render.{c,h}`** — split layout: Wi-Fi list on the left, hotspot panel on the right with SSID/password text inputs, band selector (Auto/2.4GHz/5GHz), Start/Stop button, and (when active) the join-QR. Tracks `UiFocus` (none/filter/ssid/password) and routes `SDL_TEXTINPUT` to the focused buffer.
- **`wifi/main.c`** — event loop, drives state + table refresh on action results, prefills the form from the active hotspot's settings on refresh (so editing reflects what's running), and uses a `sticky_msg` with a timeout so action results aren't immediately overwritten by the passive status line.

### Event loop contract (the important glue)

Both apps use the same pattern: `ui_handle_event` returns a small integer that `main.c` interprets. This is the only way the UI layer asks for state changes — keep the return-code table in the relevant `ui_render.h` in sync when adding codes.

**Port Commander** (src/ui_render.h:36-41):

| return | meaning                                           | main's response |
|--------|---------------------------------------------------|-----------------|
| 0      | nothing                                           | —               |
| 1      | quit                                              | exit loop       |
| 2      | refresh requested (F5)                            | re-run `lsof`   |
| 3      | filter text changed                               | rebuild `visible[]`, clear selection |
| 4      | selection changed                                 | reload `ProcessDetail` |
| 5      | user confirmed kill (second click on Kill button) | `kill(pid, SIGTERM)`, then refresh |

Two-step kill is enforced by `Ui.kill_confirm_pid`: the first click sets it to the selected PID, the second (return code 5) only fires `SIGTERM` if `pid == ui.kill_confirm_pid` (src/main.c:170-183).

**Wi-Fi Commander** (wifi/ui_render.h:46-51):

| return | meaning                | main's response |
|--------|------------------------|-----------------|
| 0      | nothing                | —               |
| 1      | quit                   | exit loop       |
| 2      | refresh requested      | re-run `wifi_state_refresh` + `wifi_table_refresh` |
| 3      | filter text changed    | rebuild `visible[]`, clear selection |
| 4      | selection changed      | (reserved — currently no per-row detail load) |
| 5      | start hotspot          | `nmcli_hotspot_up` with `ui.ssid_input` / `ui.password_input` / `ui.band` |
| 6      | stop hotspot           | `nmcli_hotspot_down(state.hotspot_conn)` |

Hotspot actions are not gated by a two-step confirm (unlike kill) — both are reversible, and typing a password is itself a deliberate action.

### Conventions worth knowing

- **No privilege escalation.** None of the three binaries invoke `sudo` themselves. `portcommander` users who need other users' sockets run that binary (or apcommander) under sudo; `wificommander` relies on the user's PolicyKit/D-Bus permissions for NetworkManager. The dashboard is a thin launcher and inherits the user's privileges as-is. Keep it that way.
- **Dashboard binary discovery.** `apcommander` resolves its own directory via `readlink("/proc/self/exe")` and looks for siblings — it does not do `$PATH` lookup, so the three binaries must live next to each other (the build directory satisfies this). If you reorganize install layout, update `self_dir()` in `dash/main.c`.
- **SIGTERM only** in `portcommander`. The kill path sends `SIGTERM`, never `SIGKILL`. Don't add a "force kill" without an explicit ask.
- **Never build shell command strings from user input.** `wificommander` accepts SSID and password from text fields — these are passed through `execvp` argv arrays in `nmcli_action.c`, never concatenated into a `popen` string. `nmcli_query.c`'s queries use static argv (no user input), but they still go through `fork`/`exec` rather than `popen` to keep the pattern consistent and the parsing safe.
- **Font discovery is a hardcoded fallback list** in each `main.c` (DejaVu → Liberation → Noto). Add new candidates to that array rather than introducing a config file. Both apps share the same list — keep them in sync if you change it.
- **Fixed-size char buffers everywhere** (`PORT_ROW_*_MAX`, `WIFI_*_MAX`, `UI_*_MAX`). Use `snprintf` and respect the existing limits rather than switching individual fields to dynamic allocation.
- **C11, no external deps beyond SDL2/SDL2_ttf, plus the runtime binaries (`lsof`/`ps` for portcommander, `nmcli` for wificommander).** Don't pull in new libraries casually — the point of this project is to stay small and apt-installable.

---
> Source: [lahirunirmalx/port-commander](https://github.com/lahirunirmalx/port-commander) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-25 -->
