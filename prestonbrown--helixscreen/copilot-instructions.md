## helixscreen

> **HelixScreen**: LVGL 9.5 touchscreen UI for Klipper 3D printers. XML engine in `lib/helix-xml/` (extracted from LVGL). Pattern: XML → Subjects → C++.

# CLAUDE.md

## Quick Start

**HelixScreen**: LVGL 9.5 touchscreen UI for Klipper 3D printers. XML engine in `lib/helix-xml/` (extracted from LVGL). Pattern: XML → Subjects → C++.

**Before compiling:** Check for existing build processes (`pgrep -f 'make|c\+\+'`) — concurrent compilations thrash the machine.

```bash
make -j                              # Build ONLY the program binary (NOT tests)
./build/bin/helix-screen --test -vv  # Mock printer + DEBUG logs
# ALWAYS use verbosity: -v=INFO, -vv=DEBUG, -vvv=TRACE (default=WARN)

make test                            # Build tests only (does NOT run them)
make test-run                        # Build AND run tests in parallel
./build/bin/helix-tests "[tag]"      # Run specific test tags
make pi-test                         # Build on thelio + deploy + run

# Worktrees — MUST use for MAJOR work. Always in .worktrees/ (project root).
scripts/setup-worktree.sh feature/my-branch  # Symlinks deps, builds fast
```

**XML hot reload:** `HELIX_HOT_RELOAD=1 ./build/bin/helix-screen --test -vv` — edit XML, save, switch panels to see changes live.

**Screenshots:** Press 'S' in UI, or `./scripts/screenshot.sh helix-screen output-name [panel]`

---

## Docs (load when needed)

Full index: **`docs/devel/CLAUDE.md`** (auto-loaded when working in docs/devel/)

Most commonly needed:

| Doc | When |
|-----|------|
| `docs/devel/UI_CONTRIBUTOR_GUIDE.md` | UI/layout work: breakpoints, tokens, colors, widgets, layout overrides |
| `docs/devel/LVGL9_XML_GUIDE.md` | XML layouts, widgets, bindings, observer cleanup |
| `docs/devel/MODAL_SYSTEM.md` | Modal architecture: ui_dialog, modal_button_row, Modal pattern |
| `docs/devel/FILAMENT_MANAGEMENT.md` | AMS, AFC, Happy Hare, ACE, AD5X IFS, CFS, Tool Changer |
| `docs/devel/ENVIRONMENT_VARIABLES.md` | Runtime env vars, mock config |
| `docs/devel/LOGGING.md` | spdlog levels: info vs debug vs trace |
| `docs/devel/BUILD_SYSTEM.md` | Makefile, cross-compilation |

---

## Code Standards

| Rule | ❌ WRONG | ✅ CORRECT |
|------|----------|-----------|
| **spdlog only** | `printf()`, `cout`, `LV_LOG_*` | `spdlog::info("temp: {}", t)` |
| **SPDX headers** | 20-line GPL boilerplate | `// SPDX-License-Identifier: GPL-3.0-or-later` |
| **RAII widgets** | `lv_malloc()` / `lv_free()` | `lvgl_make_unique<T>()` + `release()` |
| **Class-based** | `ui_panel_*_init()` functions | Classes: `MotionPanel`, `WiFiManager` |
| **Observer factory** | Static callback + `lv_observer_get_user_data()` | `observe_int_sync<Panel>()` from `observer_factory.h` |
| **Icon sync** | Add icon, forget fonts | codepoints.h + `make regen-fonts` + rebuild |
| **Formatting** | Manual formatting | Let pre-commit hook (clang-format) fix |
| **No auto-mock** | `if(!start()) return Mock()` | Check `RuntimeConfig::should_mock_*()` |
| **JSON include** | `#include <nlohmann/json.hpp>` | `#include "hv/json.hpp"` (libhv's bundled version) |
| **Build system** | `cmake`, `ninja` | `make -j` (pure Makefile) |
| **Bug commits** | `fix: thing` (no reference) | `fix(scope): thing (prestonbrown/helixscreen#123)` |

**ALWAYS:** Search the SAME FILE you're editing for similar patterns before implementing.

---

## CRITICAL RULES - Declarative UI

**DATA in C++, APPEARANCE in XML, Subjects connect them.**

| # | Rule | ❌ NEVER | ✅ ALWAYS |
|---|------|----------|----------|
| 1 | **NO lv_obj_add_event_cb()** | `lv_obj_add_event_cb(btn, cb)` | XML `<event_cb trigger="clicked" callback="name"/>` + `lv_xml_register_event_cb()` |
| 2 | **NO imperative visibility** | `lv_obj_add_flag(obj, HIDDEN)` | XML `<bind_flag_if_eq subject="state" flag="hidden" ref_value="0"/>` |
| 3 | **NO lv_label_set_text** | `lv_label_set_text(lbl, val)` | Subject binding: `<text_body bind_text="my_subject"/>` |
| 4 | **NO C++ styling** | `lv_obj_set_style_bg_color()` | XML: `style_bg_color="#card_bg"` |
| 5 | **NO manual LVGL cleanup** | `lv_display_delete()`, `lv_group_delete()` | Just `lv_deinit()` - handles everything |
| 6 | **bind_style priority** | `style_bg_color` + `bind_style` | Inline attrs override - use TWO bind_styles |
| 7 | **NO ad-hoc callback guards** | `shared_ptr<bool> alive_`, `callback_guard_`, `alive_guard_` | `lifetime_.token()` + `tok.defer(...)` via `AsyncLifetimeGuard` |
| 8 | **NO lifetime_.defer from BG** | `lifetime_.defer([this](){...})` in API callbacks | `tok.defer([this](){...})` — token holds its own shared_ptr (#707) |

**Exceptions:** DELETE cleanup, widget pool recycling, chart data, animations

---

## Design Tokens (MANDATORY)

| Category | ❌ WRONG | ✅ CORRECT |
|----------|----------|-----------|
| **Colors** | `lv_color_hex(0xE0E0E0)` | `ui_theme_get_color("card_bg")` |
| **Spacing** | `style_pad_all="12"` | `style_pad_all="#space_md"` |
| **Typography** | `<lv_label style_text_font="...">` | `<text_heading>`, `<text_body>`, `<text_small>` |

Note: `ui_theme_get_color()` for tokens, `ui_theme_parse_color()` for hex strings only (NOT tokens).

---

## Threading & Lifecycle

WebSocket/libhv callbacks = background thread. **NEVER** call `lv_subject_set_*()` directly.
Use `ui_queue_update()` from `ui_update_queue.h`. Pattern: `printer_state.cpp` `set_*_internal()`

Use `ObserverGuard` for RAII cleanup. See `observer_factory.h` for `observe_int_sync`, `observe_int_async`, `observe_string`, `observe_string_async`.

**Observer safety:** `observe_int_sync` and `observe_string` **defer callbacks** via `ui_queue_update()` to prevent re-entrant observer destruction crashes (issue #82). Use `observe_int_immediate` / `observe_string_immediate` ONLY if you're certain the callback won't modify observer lifecycle (no reassignment, no widget destruction).

**UpdateQueue ScopedFreeze (MANDATORY for drain+destroy):** When destroying widgets that may have pending deferred callbacks, always freeze the queue around the drain+destroy sequence. This closes the race window where the WebSocket background thread can enqueue new callbacks between `drain()` and widget destruction.

```cpp
auto freeze = helix::ui::UpdateQueue::instance().scoped_freeze();
helix::ui::UpdateQueue::instance().drain();
lv_obj_clean(container);  // or safe_delete(), lv_obj_delete()
// freeze thaws when it goes out of scope
```

**Async callback safety (MANDATORY):** When background threads (WebSocket, HTTP, timers) need
to update UI, use `AsyncLifetimeGuard` to prevent use-after-free if the owning object is dismissed.

`Modal` and `OverlayBase` provide `lifetime_` automatically. Standalone classes add their own:
`helix::AsyncLifetimeGuard lifetime_;`

```cpp
// Common pattern: bg thread → deferred UI update
// IMPORTANT: Use tok.defer() (NOT lifetime_.defer()) from BG-thread callbacks.
// lifetime_.defer() accesses this->lifetime_ which is a TOCTOU race — this
// can be destroyed between the tok.expired() check and the defer() call (#707).
auto tok = lifetime_.token();
api->fetch([this, tok]() {
    if (tok.expired()) return;         // Owner dismissed — skip
    tok.defer([this]() {               // Safe: uses token's own shared_ptr
        update_ui();
    });
});

// lifetime_.defer() is safe ONLY from the main thread (this guaranteed valid):
void MyPanel::on_activate() {
    lifetime_.defer([this]() { rebuild_layout(); });  // OK — main thread
}

// Cancel-and-retry (e.g., re-test while previous test in flight):
lifetime_.invalidate();                // Expire all outstanding tokens
auto tok = lifetime_.token();          // Fresh token for new operation
```

Do **NOT** use `shared_ptr<bool> alive_`, `callback_guard_`, `alive_guard_`, `weak_ptr<bool>`,
or `shared_ptr<atomic<bool>>` for callback safety. These are deprecated patterns replaced by
`AsyncLifetimeGuard`. See `include/async_lifetime_guard.h` and `docs/devel/ARCHITECTURE.md`.

**HTTP work runs on HttpExecutor, NOT raw `std::thread`.** Two process-wide lanes:
`HttpExecutor::fast()` (4 workers) for REST/API/timelapse/thumbnails/small uploads,
`HttpExecutor::slow()` (1 worker) for large file transfers. Submitted lambdas run on a
worker thread — callbacks still need `ui_queue_update()` / `tok.defer()` for UI work.
Never spawn a raw `std::thread` for HTTP — unbounded spawning crashed with EAGAIN under
thread exhaustion on RatOS (#811-adjacent). See `docs/devel/MOONRAKER_ARCHITECTURE.md`
§ "HTTP Work Execution (HttpExecutor)".

**No sync widget deletion in queued callbacks (MANDATORY):** Never call these synchronous deletion APIs from inside `queue_update()` / `async_call()` / `lifetime_.defer()` / `tok.defer()` / observer callbacks (`observe_int_sync`, `observe_string` — also deferred via queue_update since #82):

| ❌ BANNED inside queued callbacks | ✅ USE INSTEAD |
|-----------------------------------|-----------------|
| `safe_delete(ptr)` | `safe_delete_deferred(ptr)` |
| `lv_obj_delete(obj)` | `lv_obj_delete_async(obj)` |
| `lv_obj_clean(container)` | `helix::ui::safe_clean_children(container)` |

Multiple sync deletions in the same `UpdateQueue::process_pending()` batch corrupt LVGL's global event linked list → SIGSEGV in `lv_event_mark_deleted` (#776, #190, #80).

**`lifetime_.defer` / `tok.defer` do NOT escape the batch.** They are thin wrappers around `queue_update` — the callback fires in the *next* `process_pending` tick, which is still a UpdateQueue batch that may contain other sync deletions. The generation guard protects against use-after-free of `this`, NOT against event-list corruption. If you see a comment claiming `lifetime_.defer` "runs outside process_pending", it's wrong — fix it.

**Safe escape routes (truly outside UpdateQueue batches):** `safe_delete_deferred()`, `safe_delete_deferred_raw()`, `helix::ui::safe_clean_children()`, `lv_obj_delete_async()`, and raw `lv_async_call(cb, ud)`. Note: our wrapper `helix::ui::async_call` does NOT escape — it routes through `queue_update`. See `include/ui_utils.h` and `ARCHITECTURE.md` § "No safe_delete() Inside UpdateQueue Callbacks".

**Subject shutdown safety (MANDATORY):** Any class that creates LVGL subjects MUST self-register its cleanup inside `init_subjects()`. This prevents shutdown crashes (observer removal on freed subjects during `lv_deinit`). See `static_subject_registry.h` for full docs.

```cpp
void MyState::init_subjects() {
    if (subjects_initialized_) return;
    // ... create subjects ...
    subjects_initialized_ = true;
    StaticSubjectRegistry::instance().register_deinit(
        "MyState", []() { MyState::instance().deinit_subjects(); });
}
```

**Never** register cleanup externally (e.g., in SubjectInitializer). Co-locating init+cleanup prevents forgotten registrations that cause shutdown crashes.

**Dynamic subject lifetime safety (MANDATORY):** Per-fan, per-sensor, and per-extruder subjects are **dynamic** — they can be destroyed and recreated during reconnection/rediscovery. Observing a dynamic subject without a `SubjectLifetime` token causes **use-after-free crashes** when `lv_subject_deinit()` frees observers but `ObserverGuard` still holds a dangling pointer.

| ❌ CRASH | ✅ SAFE |
|----------|---------|
| `auto* s = state.get_fan_speed_subject(name);` | `SubjectLifetime lt;` |
| `obs = observe_int_sync(s, this, handler);` | `auto* s = state.get_fan_speed_subject(name, lt);` |
| | `obs = observe_int_sync(s, this, handler, lt);` |

**Dynamic subject sources** (always require lifetime token when observing):
- `PrinterFanState::get_fan_speed_subject(name, lifetime)` — per-fan speeds
- `TemperatureSensorManager::get_temp_subject(name, lifetime)` — per-sensor temps
- `PrinterTemperatureState::get_extruder_temp_subject(name, lifetime)` / `get_extruder_target_subject(name, lifetime)` — per-extruder temps

**Static subjects** (singleton lifetime, no token needed): `get_fan_speed_subject()` (no args), `get_bed_temp_subject()`, etc.

**SubjectLifetime reset ordering (MANDATORY):** When rebinding observers to new subjects, reset the lifetime BEFORE the observer. The observer guard's `weak_ptr` only expires if the `shared_ptr` (SubjectLifetime) is destroyed first. Wrong order = `lv_observer_remove()` on a freed subject (#705).

```cpp
// ✅ CORRECT: lifetime first, then observer
speed_lifetime_.reset();
speed_observer_.reset();

// ❌ CRASH: observer tries lv_observer_remove() while weak_ptr is still alive
speed_observer_.reset();
speed_lifetime_.reset();
```

See `ui_observer_guard.h` for full documentation of the `SubjectLifetime` pattern.

**No `lv_obj_delete()` in input event handlers:** Never delete container children synchronously inside `LV_EVENT_CLICKED`/`LV_EVENT_RELEASED` handlers — LVGL may be iterating the child list during `indev_proc_release`. If a rebuild (`lv_obj_clean`) follows, just null pointers and let the rebuild handle deletion. See `docs/devel/ARCHITECTURE.md` § "No Object Deletion During Input Event Processing".

---

## Patterns

| Pattern | Key Point |
|---------|-----------|
| Subject init order | Register components → init subjects → create XML |
| Widget lookup | `lv_obj_find_by_name()` not `lv_obj_get_child()` |
| Overlays | `ui_nav_push_overlay()`/`ui_nav_go_back()` |
| Modals (simple) | `Modal::show("component_name")` / `Modal::hide(dialog)` |
| Modals (subclass) | Extend `Modal`, implement `get_name()` + `component_name()`, override `on_ok()`/`on_cancel()` |
| Confirmation dialog | `modal_show_confirmation(title, msg, severity, btn_text, on_confirm, on_cancel, data)` (in `helix::ui`) |
| Modal buttons (XML) | `<modal_button_row primary_text="Save" primary_callback="on_save"/>` |

---

## Where Things Live

**Singletons** (all `::instance()`):
`PrinterState` (all printer data/subjects), `SettingsManager` (persistent settings), `NavigationManager` (panel/overlay stack), `UpdateQueue` (thread-safe UI updates), `SoundManager`, `DisplayManager`, `ModalStack`, `PrinterDetector` (printer DB + capabilities), `ToolState` (multi-tool tracking), `AmsState` (multi-backend filament systems)

**Entry flow**: `main.cpp` → `Application` → `DisplayManager` → panels via `NavigationManager`

**Key directories**:
| Path | Contents |
|------|----------|
| `src/ui/` | All UI code — flat dir, prefixed: `ui_panel_*.cpp`, `ui_overlay_*.cpp`, `ui_modal*.cpp` |
| `src/ui/modals/` | Additional modal implementations |
| `src/printer/` | PrinterState, MoonrakerAPI, macro/filament managers |
| `src/system/` | Config, settings, update checker, sound, telemetry |
| `src/application/` | App lifecycle, display, input, runtime config |
| `ui_xml/` | All XML layouts (loaded at runtime — no rebuild needed) |
| `ui_xml/components/` | Reusable XML components |
| `assets/` | Fonts, images, sounds, printer DB JSON |
| `config/` | Default config files, env templates |

**Runtime config** (on device): `~/helixscreen/config/` — settings.json, printer_database.json, helixscreen.env

**Mock-facing interfaces**: `IMoonrakerAPI` (`include/i_moonraker_api.h`) and `helix::IMoonrakerClient` (`include/i_moonraker_client.h`) are narrow pure-virtual interfaces mirroring the currently-virtual methods on `MoonrakerAPI` / `helix::MoonrakerClient`. Concrete classes inherit the interfaces; mocks still inherit concretes. Drift protection in `tests/unit/test_interface_drift_*.cpp` (`[compile][drift]` tag). Callers continue to use the concrete types — interfaces exist to enforce mock-parity at build time, not to drive call-site migration.

**Test isolation**: `HelixTestFixture` (`tests/helix_test_fixture.h`) is the base for every test fixture. Ctor + dtor call `reset_all()` which drains `UpdateQueue`, resets `SystemSettingsManager` language, clears `ModalStack`. `LVGLTestFixture` inherits it. `XMLTestFixture` owns per-instance `PrinterState` / `MoonrakerClient` / `MoonrakerAPI` (no more static test state). XML subjects still register into LVGL's global scope — per-test scopes were blocked by LVGL internals; subjects are refreshed by each test's `init_subjects(true)`.

---

## Debugging

**NEVER debug without flags!** Use `-vv` minimum.
Trust debug output. Impossible values = bug is UPSTREAM. Ask "what ELSE?" not "did first fix work?"

**Debug bundles**: `./scripts/debug-bundle.sh <SHARE_CODE> --save` to download. Save to `/tmp/` for investigation (not in repo).

---

## Critical Paths (always MAJOR work)

PrinterState, WebSocket/threading, shutdown, DisplayManager, XML processing

---

## Autonomous Sessions

When given autonomous control, Claude works independently to improve HelixScreen with minimal interruption.

**Scratchpad**: `.claude/scratchpad/` - Claude's workspace for:
- Ideas and feature concepts
- Research notes and findings
- Work-in-progress designs
- Lessons learned (like `animated_value_use_cases.md`)

**Mission**: Make HelixScreen the best damn touchscreen UI for Klipper printers.

**Autonomy Guidelines**:
- Work independently on improvements that align with existing patterns
- Commit working code with tests (don't leave broken state)
- Ask user ONLY for: major architectural decisions, UX preference calls, or when truly blocked
- Document findings in scratchpad for future sessions
- Small failures are fine - learn and move on

**Good autonomous work**:
- Polish and micro-improvements
- Code cleanup and consistency
- Adding missing tests
- Fixing obvious bugs
- Implementing features from ROADMAP.md

**Ask first**:
- New architectural patterns
- Removing/deprecating features
- Changes to critical paths (see above)
- Anything that changes user-facing behavior significantly

---
> Source: [prestonbrown/helixscreen](https://github.com/prestonbrown/helixscreen) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-21 -->
