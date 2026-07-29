## proteinview

> **Analysis Date:** 2026-03-06

# Coding Conventions

**Analysis Date:** 2026-03-06

## Naming Patterns

**Files:**
- Use `snake_case.rs` for all source files (e.g., `src/render/framebuffer.rs`, `src/model/secondary.rs`)
- Module barrel files are always `mod.rs` (e.g., `src/model/mod.rs`, `src/render/mod.rs`)
- One primary concept per file -- file name matches its main struct or function group

**Functions:**
- Use `snake_case` for all functions: `render_header`, `load_structure`, `analyze_interface`
- Public constructors use `new()` or `new_*()` pattern: `ColorScheme::new()`, `ColorScheme::new_interface()`, `Framebuffer::new()`
- State cycling methods use `cycle_*()`: `cycle_color()`, `cycle_viz_mode()`
- Toggle methods use `toggle_*()`: `toggle_interface()`
- Rendering entry points use `render_*()`: `render_header()`, `render_viewport()`, `render_helpbar()`
- Parsing functions use `parse_*()` or `load_*()`: `parse_ss_records()`, `load_structure()`
- Conversion functions use `*_to_*()`: `color_to_rgb()`, `framebuffer_to_widget()`, `to_rgb_image()`
- Internal helpers with `_fb` suffix for framebuffer-targeting variants: `render_backbone_fb()`, `render_wireframe_fb()`

**Variables:**
- Use `snake_case` for all local variables
- Abbreviated names acceptable for math-heavy code: `dx`, `dy`, `dz`, `px`, `py`, `cx`, `cy`, `r_sq`
- Loop indices commonly `i`, `j`, `n`, `seg`, `sub`
- Floating-point parameters and temporaries: `half_w`, `half_h`, `t`, `dt`

**Types:**
- Use `PascalCase` for all types: `Protein`, `Camera`, `Framebuffer`, `RibbonTriangle`
- Enums use `PascalCase` variants: `VizMode::Backbone`, `SecondaryStructure::Helix`, `ColorSchemeType::BFactor`
- Type aliases use `PascalCase`: `type V3 = [f64; 3]` (in `src/render/ribbon.rs`)

**Constants:**
- Use `SCREAMING_SNAKE_CASE`: `SPLINE_SUBDIVISIONS`, `COIL_SEGMENTS`, `HELIX_HALF_WIDTH`
- Associated constants on structs use same convention: `Camera::ROT_STEP`, `Camera::ZOOM_STEP`

## Code Style

**Formatting:**
- No `.rustfmt.toml` or `rustfmt.toml` present -- uses default `rustfmt` settings
- Standard 4-space indentation
- Run `cargo fmt` before committing

**Linting:**
- No `clippy.toml` present -- uses default Clippy settings
- `#[allow(dead_code)]` used sparingly: only on `SecondaryStructure::Turn` in `src/model/protein.rs:41`
- Run `cargo clippy` to check for lint issues

**Line Length:**
- No enforced limit, but lines generally stay under 120 characters
- Long expressions are broken across multiple lines with trailing operators

**Trailing Commas:**
- Consistently used in struct literals, function arguments, and match arms

## Import Organization

**Order:**
1. `std` library imports (`std::collections::HashSet`, `std::io`, `std::time::Duration`)
2. External crate imports (`anyhow::Result`, `ratatui::*`, `crossterm::*`, `image::*`)
3. Internal crate imports (`crate::model::protein::Protein`, `crate::render::camera::Camera`)

**Style:**
- Group related imports with nested braces: `use crossterm::{event::KeyCode, execute, terminal::{...}};`
- Use `ratatui::prelude::*` for common ratatui types (only in `src/main.rs`)
- Prefer explicit imports elsewhere: `use ratatui::style::{Color, Style, Modifier};`

**Path Aliases:**
- No path aliases configured. All imports use full `crate::` paths for internal modules
- `use super::*` used in test modules to import parent scope

## Error Handling

**Library Errors:**
- Use `anyhow::Result` as the primary error type for all fallible public functions
- `anyhow::bail!()` for early-return error conditions: `src/parser/fetch.rs:19`
- `anyhow::anyhow!()` for wrapping external errors: `src/parser/pdb.rs:16`
- Chain errors with `?` operator throughout

**Recoverable Failures:**
- Use `Option<T>` return + `?` operator for parsing functions that may fail on individual records: `parse_struct_conf_row()`, `parse_sheet_range_row()` in `src/model/secondary.rs`
- Silent fallback with `unwrap_or()` / `unwrap_or_default()` / `unwrap_or_else()` for non-critical failures:
  - `crossterm::terminal::size().unwrap_or((80, 24))` in `src/main.rs:89`
  - `pdb.identifier.as_deref().unwrap_or("Unknown")` in `src/parser/pdb.rs:48`
  - `Picker::from_query_stdio().unwrap_or_else(|_| Picker::halfblocks())` in `src/main.rs:110-111`

**Parsing Resilience:**
- Two-pass fallback strategy for structure loading: try default strictness, fall back to loose + atomic-coords-only (`src/parser/pdb.rs:9-16`)
- PDB secondary structure first, CIF secondary structure as fallback (`src/parser/pdb.rs:53-65`)
- `continue` on unparseable lines rather than aborting (`src/model/secondary.rs:33-34`)

**Panics:**
- Custom panic hook installed in `src/main.rs:93-98` to restore terminal state
- `expect()` used only for truly impossible failures: `"cannot create log file"` in `src/main.rs:85`
- `unwrap()` used in test code and for infallible conversions like `char::from_u32()` with known-valid values

## Logging

**Framework:** Custom file-based debug logging via a `log!` macro in `src/main.rs:23-31`

**Patterns:**
- Logging is opt-in via `--log <file>` CLI flag
- The `log!` macro takes an `Option<File>` and is a no-op when `None`
- Log messages are plain text with no structured format
- Log at key lifecycle points: terminal size, picker protocol, app creation, key events, frame renders
- Use conditional logging (every Nth frame) for high-frequency events: `src/main.rs:188-190`

## Comments

**When to Comment:**
- Doc comments (`///`) on all public structs, enums, and functions
- Module-level doc comments (`//!`) for complex modules: `src/render/ribbon.rs:1-10`
- Inline comments for non-obvious math, algorithm steps, and PDB format column positions
- Section separators using `// ---------------------------------------------------------------------------` in larger files: `src/render/ribbon.rs`

**Doc Style:**
- Use `///` for public items, plain `//` for private explanation
- First line is a concise summary, followed by blank line, then details
- Include format specifications in doc comments for parsers: `src/model/secondary.rs:18-21`
- Use markdown code blocks in doc comments for display format examples: `src/model/interface.rs:149-153`

**No JSDoc/TSDoc** -- this is a Rust codebase; use `///` doc comments exclusively.

## Function Design

**Size:**
- Keep functions focused on a single task
- Largest functions are rendering entry points (~40-60 lines of logic)
- Extract helpers for repeated patterns: `draw_thick_line()`, `to_pixel()`, `color_to_rgb()`

**Parameters:**
- Pass references (`&Protein`, `&Camera`, `&ColorScheme`) for read-only data
- Use `&mut` only when mutation is required: `&mut Framebuffer`, `&mut Protein`
- Pass primitives by value: `f64`, `usize`, `[u8; 3]`
- Group related parameters into structs only when they are reused (e.g., `Triangle`, `SplinePoint`)

**Return Values:**
- Use `-> Result<T>` with `anyhow` for fallible operations
- Use `-> Option<T>` for parsing that may not produce a value
- Use `-> Vec<T>` for collection-producing functions
- Return borrowed data with lifetimes when the caller does not need ownership: `pub fn ribbon_mesh(&mut self) -> &[RibbonTriangle]`

## Module Design

**Exports:**
- Each module `mod.rs` uses `pub mod` to re-export submodules
- No barrel re-exports (no `pub use` in `mod.rs` files) -- callers use full paths like `crate::model::protein::Protein`
- Public API surface is minimal: only types and functions needed by other modules are `pub`

**Barrel Files:**
- `mod.rs` files contain only `pub mod` declarations, no logic
- Example from `src/model/mod.rs`:
  ```rust
  pub mod interface;
  pub mod protein;
  pub mod secondary;
  ```

**Visibility:**
- Default to private; use `pub` only for cross-module access
- Use `pub(crate)` implicitly (fields are `pub` on structs used across modules)
- Test-only functions gated with `#[cfg(test)]`: `Framebuffer::clear()`, `Framebuffer::rasterize_triangle()`, `Framebuffer::draw_circle()`, `framebuffer_to_widget()` in `src/render/framebuffer.rs`

## Struct Patterns

**Data Structs:**
- Derive `Debug, Clone` on all model types: `Protein`, `Chain`, `Residue`, `Atom`
- Derive `Debug, Clone, Copy, PartialEq` on enums: `SecondaryStructure`, `VizMode`, `ColorSchemeType`
- All fields `pub` on model/data structs for direct access
- Mix of `pub` and private fields on stateful structs: `App` has `mesh_dirty` as private

**Enum Cycling:**
- Enums that cycle through variants implement a `next(&self) -> Self` method
- Enums that have display names implement a `name(&self) -> &str` method
- Pattern used by: `VizMode`, `ColorSchemeType`

## Performance Annotations

**Inlining:**
- Use `#[inline]` on small, hot-path functions: `set_pixel()`, `dist_sq()`, `to_pixel()`, `v3_*()` helpers
- Vector math helpers in `src/render/ribbon.rs` all annotated `#[inline]`

**Constants:**
- Associated constants preferred over module-level constants for type-specific values: `Camera::ROT_STEP`
- Module-level constants for shared numeric parameters: `SPLINE_SUBDIVISIONS`, `COIL_RADIUS`

---

*Convention analysis: 2026-03-06*

---
> Source: [001TMF/ProteinView](https://github.com/001TMF/ProteinView) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-26 -->
