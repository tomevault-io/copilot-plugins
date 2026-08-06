## dioxus-icons

> - Import icons from `dioxus_icons::lucide::*`.

# dioxus-icons Agent Notes

- Import icons from `dioxus_icons::lucide::*`.
- Use one component per icon: `rsx! { Trash { size: 20 } }`.
- Props are `size`, `color`, `stroke_width`, `stroke_linecap`, `stroke_linejoin`, and `class`.
- `size` controls both SVG width and height.
- `color` maps to SVG `stroke`; it defaults to `currentColor`.
- `stroke_width`, `stroke_linecap`, and `stroke_linejoin` map to the matching SVG stroke attributes.
- `class` defaults to an empty string and is passed to the root SVG when set.
- Do not use a generic `Icon` component or string glyph name.
- Search terms come from Lucide sidecar JSON tags and categories.
- Generated files under `crates/dioxus-icons/src/generated/` should not be edited by hand.
- Regenerate with `cargo run -p dioxus-icons-codegen`.
- If generated output changes, inspect the Lucide version pin and manifest diff.
- Public `dioxus-icons` support starts at Dioxus 0.7.7 and follows the compatible 0.7.x line.
- Keep renderer feature selection in app crates.
- `dioxus-signals` stays a normal dependency because generated Dioxus props and RSX expansion rely on Dioxus' signal-compatible prop machinery.
- docs.rs picker/widgets are part of the published docs surface.
- Pitfall: `color` controls the SVG stroke, not fill.
- Pitfall: import `dioxus_icons::lucide::Trash`.
- Pitfall: `class` accepts a Dioxus string prop; Tailwind classes belong on `class`, not `style`.

Minimum example:

```rust
use dioxus::prelude::*;
use dioxus_icons::lucide::Trash;

fn DeleteButton() -> Element {
    rsx! {
        button {
            Trash { size: 16, color: "red" }
            "Delete"
        }
    }
}
```

---
> Source: [DioxusLabs/dioxus-icons](https://github.com/DioxusLabs/dioxus-icons) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-27 -->
