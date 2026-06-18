## mpltablelayers

> `mpltablelayers` adds styled overlay layers (span cells, background shading, multi-level headers) on top of matplotlib, blume, and plottable tables. Source lives in `src/mpltablelayers/`. Documentation is built with MkDocs Material + the `mkdocs-gallery` plugin.

# mpltablelayers — Claude Code Instructions

## Project overview

`mpltablelayers` adds styled overlay layers (span cells, background shading, multi-level headers) on top of matplotlib, blume, and plottable tables. Source lives in `src/mpltablelayers/`. Documentation is built with MkDocs Material + the `mkdocs-gallery` plugin.

---

## Adding new gallery examples

### Where files go

| Scenario                        | Directory                     |
|---------------------------------|-------------------------------|
| Core feature example            | `examples/`                   |
| Integration with a table backend| `examples/integrations/`      |
| Known limitation or caveat      | `examples/known_limitations/` |

The gallery plugin (`mkdocs.yml → plugins.gallery`) picks up every `.py` file under `examples/` recursively. Files are sorted within a subsection by filename (`FileNameSortKey`), so prefix with a number (`plot_0_`, `plot_1_`, …) when order matters.

### File naming convention

```text
plot_<N>_<short_slug>.py          # numbered, for ordered core examples
plot_<short_slug>.py              # un-numbered, for financial / showcase sub-galleries
```

### Required file structure

Every example file must follow this exact structure:

```python
"""# <Page title — becomes the gallery card title and the H1 heading>

<One or two sentences describing what this example demonstrates.>

<Optional extra prose, bullet lists, or admonitions.>
"""

# %%           ← mandatory cell separator before the first import
import matplotlib.pyplot as plt
import pandas as pd

import mpltablelayers

# ... plotting code ...

_ = plt.tight_layout()   # assign to _ to suppress output in gallery
```

**Rules:**

1. **Module docstring** — the very first thing in the file. The first line must be `"""# <Title>` — the `#` inside the docstring is what `mkdocs-gallery` renders as the page H1 and the gallery thumbnail caption.
2. **`# %%` cell separators** — mark the start of each executable cell. The first `# %%` goes immediately before the imports. Additional cells can be separated with `# %%` optionally followed by a plain-text comment that becomes a sub-heading.
3. **`_ = plt.tight_layout()`** — assign the return value to `_` on the last line so the gallery renderer does not print `None`.
4. **No `plt.show()`** — the gallery plugin captures figures automatically; never call `plt.show()`.
5. **`figsize`** — use `(8, 5)` for a single table; `(10, 6)` for side-by-side comparisons.

### Docstring prose style

- First line after the title: one declarative sentence stating the feature.
- Use backtick-quoted parameter names: `` ``bushours`` ``, `` ``weekmask`` ``, `` ``holidays`` ``.
- Keep the "Core code" block to the single most important `set_xscale` call.
- Sub-sections in the docstring use RST-style headings (`Workarounds\n-----------`) only for known-limitations files.


## MkDocs / gallery build

```bash
mkdocs serve          # live preview
mkdocs build          # static build into site/
```

The gallery plugin writes generated Markdown to `docs/generated/gallery/` — do not edit those files by hand.


## Regenerating example PNGs

`examples/run_all.py` executes every `plot_*.py` under `examples/` with the Agg
backend and writes PNGs to `examples/output/` (gitignored). Use it when you
want to eyeball the gallery without building the full MkDocs site.

```bash
uv run --with blume --with plottable python examples/run_all.py
```

`blume` and `plottable` are **optional** runtime deps for
`plot_1_blume_shading.py` and `plot_2_plottable_shading.py` respectively; they
are intentionally *not* in `pyproject.toml`, so pass them via `--with` (or
install them ad-hoc).


## API gotchas

Things that are easy to get wrong when composing tables with this package:

- **`cell.visible_edges = ""` kills the fill, not just the border.** Matplotlib's
  `Cell.get_path()` emits all-MOVETO codes when no edges are visible, so the
  rectangle is never closed and therefore not filled. To hide a cell's border
  while keeping its background color, use
  `cell.set_linewidth(0); cell.set_edgecolor("none")`.

- **`add_table_multispan_cell(y0x0=...)` anchors at the bottom-left**, and
  `height` grows *upward* (matplotlib table convention: smaller row indices
  are drawn higher). To cover data rows 3–7 with a background, pass
  `y0x0=(7, col), height=5`, not `y0x0=(3, col), height=5`.

- **`add_table_multispan_cell` defaults to `zorder=-1`** (background shading).
  For spans that should paint *over* underlying cells (banners, category
  headers, totals rules), pass `zorder=2` or higher explicitly. The same
  applies when forwarding `**props` through `add_hierarchical_header` — add
  `"zorder": 2` to `default_properties` if the underlying cells have any
  non-transparent fill.

---
> Source: [saemeon/mpltablelayers](https://github.com/saemeon/mpltablelayers) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
