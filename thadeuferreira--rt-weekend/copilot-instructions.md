## rt-weekend

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build & Run

This project is written in the [Odin programming language](https://odin-lang.org/).

**Build (Makefile):**

```bash
make debug    # Debug build → build/debug (-define:VERBOSE_DEBUG=1)
make release  # Release build → build/release (-o:speed -no-bounds-check -define:PROFILING_ENABLED=false -define:VERBOSE_OUTPUT=false -define:TRACE_CAPTURE_ENABLED=false)
```

**Manual build:**

```bash
# Debug
odin build . -collection:RT_Weekend=. -debug -define:VERBOSE_DEBUG=1 -out:build/debug

# Release (optimized, quiet stdout)
odin build . -collection:RT_Weekend=. -o:speed -no-bounds-check -define:PROFILING_ENABLED=false -define:VERBOSE_OUTPUT=false -define:TRACE_CAPTURE_ENABLED=false -out:build/release
```

**Run:**

```bash
# Run with defaults (800px wide, 10 samples, 10 spheres)
./build/debug
# or
./build/release

# Custom resolution / samples / threads
./build/debug -w 400 -h 225 -s 20 -c 8

# Full options
./build/debug -w <int> -h <int> -s <samples> -n <spheres> -c <threads>
```

**VERBOSE_DEBUG:** Convenience compile-time flag (`#config`, default `0`). Use `-define:VERBOSE_DEBUG=1` to enable diagnostics defaults in one switch:
- `TRACK_ALLOCATIONS` default becomes enabled
- `UI_EVENT_LOG_ENABLED` default becomes enabled
- `EDITOR_IDLE_GPU_TRACE` default becomes enabled

Individual flags can still be overridden explicitly with their own `-define`.  
Runtime override for idle GPU tracing: `EDITOR_IDLE_GPU_TRACE=0|1` (`0/off/false` disables, `1/on/true` enables).

**VERBOSE_OUTPUT:** A compile-time flag (`#config`, default `true`) controls non-essential stdout: startup config, system info, per-thread stats, timing breakdown, and GPU success messages. Error and fallback messages (e.g. GPU init failed) are always printed. Release builds set `VERBOSE_OUTPUT=false`.

**TRACE_CAPTURE_ENABLED:** When `true`, Chrome-trace visual benchmarking (Render → Start/Stop Visual Benchmark Capture) is available; when `false`, trace code is compiled out and the benchmark menu items are disabled. Release builds set `TRACE_CAPTURE_ENABLED=false` so capture and file I/O add no overhead.

**FILE_MODAL_FALLBACK:** When `true`, if the native file dialog is unavailable (e.g. zenity not installed), the editor falls back to the text path modal for Import and Save As. Default is `false`. Defined in `editor/core_cmd_actions.odin`.

**Key flags:**
- `-w` / `-h`: image width / height in pixels
- `-s`: samples per pixel (default 10)
- `-n`: number of small random spheres (default 10)
- `-c`: thread count (defaults to physical CPU cores)

The program opens a **Raylib window** with floating panels:
- **Viewport** — unified Editor / Raytrace view with mode toggle and render settings
- **Stats** — tile progress, thread count, elapsed time
- **Console** — startup and completion messages
- **Details** — selected object properties
- **World Outliner** — scene object list

Drag panels by their title bar; resize from the bottom-right grip.

## Dependencies

### Raylib (via Odin vendor collection)
Raylib is bundled with the Odin compiler distribution under `$(odin root)/vendor/raylib/`.
The Linux shared library (`libraylib.so`) is included in `vendor/raylib/linux/` — no separate system install is required as long as your Odin installation is complete.

If you encounter linker errors on Linux, verify that:
```bash
ls $(odin root)/vendor/raylib/linux/libraylib.so
```
exists. If not, install Odin from [odin-lang.org](https://odin-lang.org/) with the full vendor collection.

### GPU path (Vulkan compute)
The GPU compute-shader backend uses **Vulkan** via the `vk_ctx` package (`vendor:vulkan` + `vendor:glfw`). On platforms without a Vulkan driver, `create_gpu_renderer` returns nil and the renderer falls back to CPU automatically.

### Vulkan infrastructure (`vk_ctx`)
The **`vk_ctx`** package bootstraps a Vulkan instance, device, queues, command pools, compute pipeline, buffer management, and synchronization utilities. It is used by the GPU backend (`raytrace/gpu_backend_vulkan.odin`) and the standalone smoke test. Build the smoke binary from the repo root:

```bash
make vk-smoke
./build/vk_smoke --headless          # instance / device / queues / pool
./build/vk_smoke --headless --clear  # + one-shot clear-color image submit
./build/vk_smoke --window [--platform=auto|x11|wayland] [--frames=N] # present Vulkan hello-triangle
```

Requires a system **Vulkan loader** (`libvulkan`) and **GLFW** (Odin’s `vendor/glfw` links the system or static library). `vk_load_vulkan` checks `glfw.VulkanSupported()`, that `vkGetInstanceProcAddr` is non-null after loading, and that `vkEnumerateInstanceExtensionProperties` succeeds, so missing loaders/ICDs fail with an error instead of crashing later.

If the loader advertises **VK_KHR_portability_enumeration**, `create_instance` enables it and sets **VK_INSTANCE_CREATE_ENUMERATE_PORTABILITY_BIT_KHR** so portability ICDs (e.g. MoltenVK) are enumerated; **VK_KHR_portability_subset** is enabled on the logical device when the GPU supports it.

Optional **validation layers** (`vulkan-validation-layers` on many distros): enabled in debug builds when `VK_LAYER_KHRONOS_validation` is present (`-define:VK_VALIDATION=false` to force off). See `tools/vk_smoke/main.odin` for flags.

### UI assets (SDF font and shader)
- **`assets/fonts/Inter-Regular.ttf`** — Inter (OFL-licensed, Arial-like) used for UI text when SDF font loading succeeds. Sourced from [rsms/inter](https://github.com/rsms/inter).
- **`assets/shaders/sdf.fs`** — Raylib SDF fragment shader (GLSL 330) for signed-distance-field text rendering.
- **`assets/shaders/raytrace_vk.comp`** — Vulkan GLSL 450 compute shader (GPU path tracer). Pre-compiled to `raytrace_vk.comp.spv` (committed); rebuild with `make shaders`. The SPIR-V is embedded in the binary via `#load`.

**Feature flag:** SDF/custom font loading is **off by default** (`USE_SDF_FONT = false`). With it disabled, the UI uses Raylib’s default font only. To enable SDF and custom font loading, build with `-define:USE_SDF_FONT=true`.

Paths are resolved **relative to the current working directory** at launch. Run the binary from the repository root (e.g. `./build/debug`) so `assets/fonts/` and `assets/shaders/sdf.fs` are found; the default directory for Open/Save scene dialogs when no file is open is then `./scenes`. The `scenes/` folder is in the repo (with `.gitkeep`); user scene files are ignored via `.gitignore` (`scenes/*`, `!scenes/.gitkeep`). If the SDF assets are missing with the flag enabled, the UI falls back to Raylib’s default font.

## Project layout (packages)

Layout is Godot-inspired: **core** (shared types), **raytrace** (renderer only), **persistence** (scene/config load and save), **editor** (application UI). See each major folder’s **AGENTS.md** for scoped context.

- **`main.odin`** (package `main`) — entry point; imports `RT_Weekend:core`, `RT_Weekend:util`, `RT_Weekend:raytrace`, `RT_Weekend:persistence`, `RT_Weekend:editor`.
- **`core/`** (package `core`) — shared types used by editor and renderer: `MaterialKind`, `CameraParams`, `SceneSphere`, **Texture** (ConstantTexture, CheckerTexture). No I/O, no editor/raytrace dependency. See [core/AGENTS.md](core/AGENTS.md).
- **`util/`** (package `util`) — CLI args (`Args`, `parse_args_with_short_flags`), system info (`get_number_of_physical_cores`, `print_system_info`), per-thread RNG. No file I/O. See [util/AGENTS.md](util/AGENTS.md).
- **`raytrace/`** (package `raytrace`) — path tracer only: camera, BVH, materials, vector/ray math, pixel buffer, profiling, `scene_build`. No scene file I/O. See [raytrace/AGENTS.md](raytrace/AGENTS.md).
- **`persistence/`** (package `persistence`) — scene/config persistence: `load_scene`/`save_scene`, `load_config`/`save_config`; types `RenderConfig`, `EditorLayout`, etc. Depends on core and raytrace. See [persistence/AGENTS.md](persistence/AGENTS.md).
- **`editor/`** (package `editor`) — Raylib window, panels, menus, layout, widgets, fonts. Exports `run_app`. Depends on core, util, raytrace, persistence. See [editor/AGENTS.md](editor/AGENTS.md).
- **`vk_ctx/`** (package `vk_ctx`) — optional Vulkan instance/device/queues/command pool + sync; used by `tools/vk_smoke` only. Not a dependency of `raytrace` or `editor`.

## Architecture Overview

The renderer is a standard path tracer built around these layers:

### Entry & Scene Setup (`main.odin` + `raytrace/` + `persistence/`)
`main.odin` parses CLI arguments (util), loads config/scene via `persistence` when paths are given, builds camera and world (or uses raytrace for empty scene), then calls `editor.run_app()`.

Scene file I/O is in **persistence** (`load_scene`, `save_scene`); config I/O is `persistence.load_config` / `save_config`.

**Scene background fallback on load:** `persistence.load_scene` preserves the serialized camera background when it is non-black. If the loaded scene background is black/missing and the scene has no emissive materials (`diffuse_light`), it forces a white background (`{1,1,1}`) before render so unlit scenes remain visible.

### Editor (`editor/`)
`run_app()` (in `editor/app.odin`) owns the main thread and the Raylib event loop:
1. Creates the window and a GPU texture sized to the render output
2. Calls `start_render()` to kick off background worker threads (non-blocking)
3. Each frame: uploads partial pixel buffer to GPU, polls progress, calls `finish_render()` when all tiles are done
4. Draws panels, menu bar, and layout via `editor/ui_chrome.odin` and panel-specific modules

**Idle GPU fix (Editor mode):** The viewport and camera preview are now dirty-flag driven, and the Vulkan/ImGui loop uses `glfw.WaitEventsTimeout` when fully idle (`render_state=Idle`, no viewport/preview dirty flags, no active drag/input). This removes unnecessary ~60 FPS present churn while idle and lowers baseline GPU usage.

**`editor/ui_chrome.odin`**: Panel chrome and theme — `update_panel`, `draw_panel_chrome`, `upload_render_texture`; `PanelStyle` and shared constants (`TITLE_BAR_HEIGHT`, `ACCENT_COLOR`, etc.). Panel IDs (`PANEL_ID_VIEWPORT`, `PANEL_ID_STATS`, `PANEL_ID_CONSOLE`, `PANEL_ID_CAMERA`, `PANEL_ID_DETAILS`, `PANEL_ID_CAMERA_PREVIEW`, `PANEL_ID_OUTLINER`, `PANEL_ID_SYSTEM_INFO`) are defined in `app.odin`. Menus and layout live in the same package. SDF font in `editor/ui_font.odin`; `draw_ui_text` / `measure_ui_text` in `app.odin`.

**File dialogs and unsaved state:** Open/Save use **native OS dialogs** (no C/C++ libs) via `util.open_file_dialog`, `util.save_file_dialog`, and `util.dialog_default_dir`. When no scene file is open, the default directory is **`<cwd>/scenes`**; the `scenes/` folder exists in the repo (see `.gitignore`: `scenes/*` ignored, `!scenes/.gitkeep` keeps the folder). If the native dialog is unavailable, build with `-define:FILE_MODAL_FALLBACK=true` to fall back to the text path modal. **Save Changes?** appears when exiting or importing with unsaved changes; options are Save (overwrite), Save As (pick path), Cancel, Continue (discard). The editor only proceeds (exit/import) after a successful save or when the user chooses Continue/Cancel. **Load Example** always shows a confirmation; “Save & Load” is enabled only when the scene is dirty and uses Save As when there is no current path. Unsaved state is tracked in `e_scene_dirty`; any edit (including undo/redo) and all camera/material changes call `mark_scene_dirty`. `file_import_from_path` resets edit history; `cmd_action_file_save` and `file_save_as_path` return `bool` so callers can avoid proceeding on save failure.

### Non-blocking Render API (`raytrace/camera.odin`)
Three procedures replace the old blocking `render_parallel`:
- `start_render(r_camera, world, num_threads) -> ^RenderSession` — allocates buffer, builds BVH, spawns threads, returns immediately
- `get_render_progress(session) -> f32` — returns 0.0–1.0; safe to call from any thread
- `finish_render(session)` — joins threads, frees BVH + contexts, prints timing breakdown to stdout

All per-thread state lives in `RenderSession` (no package-level globals).

### Parallel Rendering (`raytrace/camera.odin`)
The rendering loop (unchanged from previous design):
1. Allocates a flat `TestPixelBuffer` (width × height × 3 floats, row-major)
2. Builds the **BVH** once from the scene geometry
3. Divides the image into **32×32 pixel tiles**
4. Spawns N worker threads; each atomically pulls tiles from a shared work queue (`sync.atomic_add`)
5. Each thread has its own **Xoshiro256++ RNG** (seeded uniquely to avoid races)
6. Threads write to non-overlapping regions of the pixel buffer — no locking needed for writes

### Acceleration Structure (`raytrace/hittable.odin`)
BVH (Bounding Volume Hierarchy) built with median splitting. Reduces ray–scene intersection from O(n) to O(log n). Constructed once, then shared read-only across all threads.

### Ray & Color Logic (`raytrace/vector3.odin`, `raytrace/raytrace.odin`)
- `raytrace/vector3.odin`: Vec3 math, ray definition, `ray_color`, `linear_to_gamma`, `infinity`
- `raytrace/raytrace.odin`: `setup_scene` builds world geometry; `write_buffer_to_ppm` saves the final image

### Materials (`raytrace/material.odin`)
Union-based dispatch over `Lambertian`, `Metallic`, and `Dielectric`. Each implements a `scatter` procedure returning the scattered ray and attenuation color. Lambertian uses **Texture** (value) for albedo; sampling is via `texture_value(tex, u, v, p)` in **raytrace/texture.odin**.

### Textures (core + raytrace)
- **core/types.odin** — `ConstantTexture` (single color), `CheckerTexture` (scale, even, odd colors), `Texture` union. **SceneSphere.albedo** is a `Texture` value.
- **raytrace/texture.odin** — Aliases from core; `texture_value(tex: Texture, u, v: f32, p: [3]f32) -> [3]f32` for CPU path. Textures are passed **by value**; prefer stack allocation over pointers.
- **raytrace/gpu_types.odin** — GPU path uses `TEX_CONSTANT` / `TEX_CHECKER` and `tex_scale`, `tex_even`, `tex_odd` in `GPUSphere`; `scene_to_gpu_spheres` converts core Texture to GPU fields.

### Geometry (`raytrace/hittable.odin`)
`Sphere` and `Quad` primitives (quad = planar rectangle via Q + αu + βv; helpers `xy_rect`, `xz_rect`, `yz_rect` for walls/floors/ceilings). BVH nodes are also `Hittable` variants (recursive union tree). Spheres support **motion blur** via `center` → `center1` over the shutter interval when `is_moving` is true.

### Motion blur (editor + core + persistence)
- **Camera shutter** — `core.CameraParams` and `rt.Camera` have `shutter_open` and `shutter_close` (normalized 0..1). The Camera panel and Details (camera selected) expose drag fields; values are copied via `apply_scene_camera` / `copy_camera_to_scene_params`. `get_ray` samples ray time with `util.random_float_range(rng, shutter_open, shutter_close)` so the shutter window affects motion blur.
- **Per-sphere movement** — In Details, when a sphere is selected, a **MOTION** section shows **dX/dY/dZ** (offset = `center1 − center`). Editing updates `center1` and sets `is_moving` when the offset is non-zero. Undo/dirty and scene export use existing `ModifySphereAction` and `SetSceneSphere`.
- **Persistence** — Scene JSON saves/loads `shutter_open`, `shutter_close` on the camera and `center1`, `is_moving` on spheres. Older files without these fields load with default shutter [0,1] and static spheres.

### Profiling (`raytrace/profiling.odin`)
Zero-cost profiling gated by `PROFILING_ENABLED` compile flag. Records nanosecond-precision timings per thread for: ray generation, `ray_color`, intersection, scatter, background, and pixel setup. When `VERBOSE_OUTPUT` is true, per-phase wall-clock breakdowns (BVH build, render, I/O) are printed to stdout after the render completes. `aggregate_into_summary` always runs so the Stats panel retains profiling data even in release mode.

### Utilities (`util/`)
- `util/cli.odin`: CLI argument parsing (`Args`, `parse_args_with_short_flags`), system info (`get_number_of_physical_cores`, `print_system_info` via `core:sys/info`).
- `util/rng.odin`: Xoshiro256++ PRNG (`ThreadRNG`, `create_thread_rng`, `random_float`, `random_float_range`). Config/file I/O lives in **persistence**, not util.
- `util/native_dialog.odin`: Cross-platform native open/save file dialogs via CLI (Linux: zenity, macOS: osascript, Windows: PowerShell). `open_file_dialog`, `save_file_dialog`, `dialog_default_dir` (returns directory of current file, or `<cwd>/scenes` when no file). No C/C++ libraries. Returned path is owned by the caller (must `delete`). Filter constants `SCENE_FILTER_DESC` / `SCENE_FILTER_EXT` for scene JSON.

### Interval Math (`raytrace/interval.odin`)
Simple `Interval` struct used for ray `t`-range clamping and AABB overlap tests.

## Allocation preference

In this project we prefer **stack allocation** as much as possible. Don’t pass pointers around unless it is strictly necessary (e.g. optional “no value”, large shared buffers, or legacy APIs). Texture, SceneSphere, and material data are value types; pass them by value.

## Odin-Specific Notes

- Odin uses `import` and has built-in `sync`, `thread`, and `fmt` packages.
- The project uses multiple packages: `main` (root), `core`, `util`, `raytrace`, `persistence`, `editor`. Build with `-collection:RT_Weekend=.` so that `import "RT_Weekend:editor"` etc. resolve.
- `sync.atomic_add`, `sync.atomic_load` are used for lock-free tile dispatch and progress tracking.
- No heap allocator is configured explicitly; Odin's default context allocator is used.
- Worker threads receive their context via `thread.Thread.data` (a `rawptr` cast to `^ParallelRenderContext`).
- The Raylib trace log callback uses `proc "c"` calling convention; `context = runtime.default_context()` is required at the top to use Odin allocators.
- **Naming (scope prefixes):** Use idiomatic Odin type/function names. Variables/fields use short scope prefixes where useful: `e_` (editor), `r_` (render), `c_` (core). See each package's AGENTS.md for the convention.

## Instrumentation (idiomatic Odin)

Odin doesn’t use C/C++-style `#define` macros for RAII timers. In this repo we follow the idiomatic pattern:

- **Compile-time flags** via `#config` (e.g. `PROFILING_ENABLED`, `TRACE_CAPTURE_ENABLED`).
- **Scope object + `defer`** so end markers fire on scope exit.

Profiling counters (for Stats panel / aggregate breakdown) are in `raytrace/profiling.odin`:

```odin
scope := PROFILE_SCOPE(&thread_breakdown.get_ray_time)
defer PROFILE_SCOPE_END(scope)
```

Chrome/Perfetto timeline tracing is in `util/trace.odin`:

```odin
s := util.trace_scope_begin("Frame", "game")
defer util.trace_scope_end(s)
```

If you need *zero* call-site overhead, wrap the whole scope in `when <FLAG> { ... }`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ThadeuFerreira) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
