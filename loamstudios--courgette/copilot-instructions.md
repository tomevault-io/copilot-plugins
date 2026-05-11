## courgette

> Project conventions and context for AI agents and contributors working in Courgette.

# AGENTS.md

Project conventions and context for AI agents and contributors working in Courgette.

## What This Is

Courgette is a declarative TUI framework for Elixir built on OTP. Zero external
dependencies (except ex_doc for docs). ~9,700 lines of Elixir across ~44 modules.

## Project Structure

```
lib/courgette.ex                    # Entry point: run/2, stop/0, send_update/2
lib/courgette/
  app.ex                            # `use Courgette.App` macro (root component)
  component.ex                      # Function component system (attrs, slots, assigns)
  component/dsl.ex                  # Macros: box, text, input, live_component, etc.
  live_component.ex                 # LiveComponent behaviour + state helpers
  live_component/server.ex          # GenServer backing each live component instance
  live_component/lifecycle.ex       # Child reconciliation (start/update/stop)
  element.ex                        # Element tree data structure (pure data)
  renderer.ex                       # Render pipeline orchestrator (GenServer, ~60fps)
  layout/
    engine.ex                       # Public API: Engine.compute/2
    engine/flex.ex                  # CSS Flexbox algorithm (15-step, Taffy-verified)
    engine/style.ex                 # Style extraction from element props
    engine/geometry.ex              # Axis-relative helpers (main/cross)
    engine/text.ex                  # Text measurement for overflow modes
    engine/round.ex                 # Float→int rounding
    bounds.ex                       # Rectangle struct + collision detection
  painter.ex                        # Layout tree → buffer painting
  buffer.ex                         # 2D cell grid
  buffer/cell.ex                    # Single cell: grapheme + colors + styles
  buffer/diff.ex                    # Front/back buffer diff → runs
  buffer/writer.ex                  # Runs → optimized ANSI escape sequences
  buffer/double_buffer.ex           # Double buffering wrapper
  terminal.ex                       # Terminal lifecycle (raw mode, alt screen, cleanup)
  terminal/key_parser.ex            # Bytes → structured events (pure function)
  terminal/signal_handler.ex        # SIGWINCH resize handler
  ansi.ex                           # ANSI escape sequence constants + builders
  ansi/color_mode.ex                # Terminal color capability detection
  theme.ex                          # Semantic color tokens
  focus_manager.ex                  # Pure focus tracking (Tab/Shift-Tab cycling)
  component_registry.ex             # ETS-backed {module, id} → pid lookup
  components.ex                     # Built-in function components (badge, heading, etc.)
  components/text_input.ex          # Single-line text input
  components/textarea.ex            # Multi-line text area
  components/select.ex              # Dropdown selector
  components/spinner.ex             # Animated loading indicator
  components/progress_bar.ex        # Progress bar with animation
  animation/easing.ex               # Easing functions (linear, ease_out, bounce, etc.)
  animation/frames.ex               # Frame cycling for spinners
  animation/tween.ex                # Value interpolation with wall clock
  component_test_helpers.ex         # Headless test helpers (mount, send_event, etc.)
test/                               # 40+ test files mirroring lib/ structure
examples/                           # Runnable examples (mix run examples/*.exs)
```

## Rendering Pipeline

```
Component render/1 → Element Tree → Layout (Flexbox) → Paint (Buffer) → Diff → ANSI → Terminal
```

This is a strict one-way data flow. Element trees are pure data. Layout
computes positions. Paint fills a buffer. Diff finds changes. Writer emits
ANSI. The Renderer GenServer orchestrates this at ~60fps with frame batching.

## Key Design Patterns

1. **Pure data, processes only where needed.** Elements, Bounds, FocusManager,
   KeyParser, Buffer, Diff — all pure functions. GenServers only for Terminal,
   Renderer, and LiveComponent.Server.

2. **Keyword-in, map-internally.** Public APIs accept keyword lists. Internally
   everything is maps. `assigns/1` converts at the component boundary.

3. **Injection for testing.** The Tween clock is injectable. Renderer has
   headless mode. ComponentTestHelpers skip the terminal entirely.

4. **CSS Flexbox correctness.** The layout engine follows the CSS spec and uses
   Taffy as a reference. `mix taffy.generate` auto-translates Taffy's Rust
   test fixtures into Elixir tests. Currently 262/262 (100%) passing.

5. **Minimal ANSI output.** Writer tracks SGR state across cells and only emits
   changes. Synchronized output markers prevent tearing.

## Code Conventions

- **Elixir ~> 1.18**, standard `mix format`
- Module docs on every module (`@moduledoc` — 100% coverage, keep it that way)
- `@doc` and `@spec` on all public functions
- Types defined with `@type` for all public structs
- No external dependencies in lib/ (ex_doc is dev-only)
- Tests mirror the lib/ directory structure
- Avoid abbreviations in public API names

## Testing

```bash
mix test                    # Run all tests
mix test test/specific_test.exs  # Run one file
```

**Test helpers pattern:**
```elixir
use Courgette.ComponentTestHelpers

view = mount(MyComponent, width: 80, height: 24, initial_assigns: %{})
send_event(view, {:key, :arrow_up})
send_tab(view)
assert render_text(view) =~ "expected text"
tree = render_tree(view)  # Get raw element tree for structural assertions
```

**Animations in tests:** Set `Application.put_env(:courgette, :animations_enabled, false)`
to skip timers. Components should check `Courgette.animations_enabled?()` in mount.

**Flex layout tests:** Generated from Taffy fixtures via `mix taffy.generate`.
Run the generator after updating flex.ex to verify correctness against Taffy.

## Common Tasks

### Adding a new built-in component

1. Create `lib/courgette/components/my_component.ex`
2. `use Courgette.LiveComponent` for stateful, `use Courgette.Component` for stateless
3. Add to the `:Components` group in `mix.exs` docs config
4. Add tests in `test/courgette/components/my_component_test.exs`

### Adding a new element type

1. Add the atom to `@valid_types` in `element.ex`
2. Add the atom to `@element_types` in `dsl.ex` (auto-generates macros)
3. Add the type to `element_type` typespec in `element.ex`
4. Handle it in `painter.ex` and `layout/engine/style.ex`

### Modifying the layout engine

1. Edit `lib/courgette/layout/engine/flex.ex`
2. Run `mix test test/courgette/layout/` to verify against Taffy fixtures
3. If adding new CSS properties, add them to `style.ex` extraction

### Adding a new event type

1. Add parsing in `terminal/key_parser.ex`
2. Add the type to `@type event()` and `@type key_value()` typespecs
3. Handle in `live_component/server.ex` if it needs special routing

## Input Event Format

Events are tuples. Common shapes:

```elixir
{:key, {:char, "a"}}           # Printable character
{:key, :enter}                  # Named key
{:key, :arrow_up}               # Arrow key
{:key, {:shift, :tab}}          # Modified key
{:key, {:ctrl, {:char, "c"}}}   # Ctrl combo
{:mouse, :press, :left, col, row, []}  # Mouse click
{:focus, :in}                   # Terminal focus
{:paste, "pasted text"}         # Bracketed paste
```

## Where Things Live

| Concept | Module(s) |
|---------|-----------|
| App entry point | `Courgette`, `Courgette.App` |
| Component definition | `Component`, `LiveComponent`, `Component.DSL` |
| Element tree | `Element` |
| Layout computation | `Layout.Engine`, `Layout.Engine.Flex` |
| Buffer + cells | `Buffer`, `Buffer.Cell` |
| Rendering pipeline | `Renderer`, `Painter`, `Buffer.Diff`, `Buffer.Writer` |
| Terminal I/O | `Terminal`, `ANSI`, `ANSI.ColorMode` |
| Input parsing | `Terminal.KeyParser` |
| Focus | `FocusManager` |
| State management | `LiveComponent` (assign/update helpers) |
| Theming | `Theme` |
| Animation | `Animation.Tween`, `Animation.Easing`, `Animation.Frames` |
| Testing | `ComponentTestHelpers` |
| Component lookup | `ComponentRegistry` (ETS) |

---
> Source: [LoamStudios/courgette](https://github.com/LoamStudios/courgette) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-05 -->
