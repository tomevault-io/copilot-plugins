## graphs

> >


# graph-design

Economist-style data-visualisation system for matplotlib and seaborn.

> **This repo is the skill** (standard agent-skill layout: `SKILL.md` +
> [`references/`](./references) + [`examples/`](./examples)) *and* the Python
> package that powers it. Install the library with `pip install
> djrhails-graphs`; install the skill by submoduling/vendoring this repo as a
> skill directory (e.g. `skills/graph-design`). Import as `graphs`.

## Quick start

A two-series line chart with direct labels, a footnote marker, and a packed
footnote-plus-source row — the conventions the library is built for.

```python
import matplotlib.pyplot as plt
import numpy as np

from graphs import finalize, footnotes, label_lines, save_chart, set_theme, subplots

set_theme()

months = np.arange(24)
us = 2.0 + 4.0 * np.exp(-months / 9) + np.random.default_rng(0).normal(0, 0.25, 24)
eu = 1.8 + 4.5 * np.exp(-months / 11) + np.random.default_rng(7).normal(0, 0.30, 24)

fig, ax = subplots("wide")
ax.plot(months, us, label="America")
ax.plot(months, eu, label="Euro area")
label_lines(ax)

finalize(
    ax,
    title="Cooling off",
    descriptor="Headline CPI*, % change on a year earlier, monthly",
    footnote_lines=1,  # reserve room for the footnote row drawn below
)
footnotes(
    fig,
    "*All-items consumer price index",
    source="Sources: [US Bureau of Labor Statistics](https://www.bls.gov/); "
           "[Eurostat](https://ec.europa.eu/eurostat)",
)

save_chart(__file__)  # writes quick.png beside the script
```

Save to `quick.py` and run; the output sits next to it. `finalize()`
auto-sizes margins, and `footnotes()` packs the note and source onto one
row when they fit, wrapping when they don't — leave `source=` off
`finalize()` so they belong to the same line, and pass `footnote_lines=`
so auto-layout reserves the bottom band for them. `save_chart` is the
standard epilogue: tight bbox, 150 dpi, close, one-line confirmation.

### Default visual conventions

Behaviour that's automatic unless you override it:

- **Charts come in two widths.** Create figures with
  `subplots("daily")` (4.6in column, portrait-leaning) or
  `subplots("wide")` (7.0in article format) — like a newspaper's column
  formats, the width is fixed by the medium and only the height is the
  per-chart choice (`height=`). Fixed widths keep the type-to-chart
  ratio consistent across a set; ad-hoc `figsize=` widths are what make
  a gallery look ragged. Extra `plt.subplots` kwargs pass through
  (`ncols=`, `sharex=`, …).
- **Title marker is the favicon triangle** (`marker="delta"`). The hollow
  red triangle is drawn inline at the first title line's baseline, sized
  to the cap height. Pass `marker="rule"` for the legacy short red rule
  above the title, or `marker="none"` to suppress entirely.
- **Titles and descriptors auto-wrap to the figure width.** Pass them as
  single lines — `finalize()` measures with the renderer and breaks where
  *this* figure needs it (with a widow fix, so no single-word last lines).
  Never copy a reference chart's line breaks; explicit `\n` is reserved
  for semantic breaks (a descriptor's subject / unit split).
- **Explicit descriptor breaks get a semibold lead.** An explicit `\n`
  splits the descriptor into a semibold subject lead over regular
  scope/unit lines; the lead stays semibold even if it wraps. Breaks
  added by the auto-wrap alone never trigger the styling.
- **Numeric y labels sit on their gridlines** (`y_labels="on_grid"`,
  the `finalize()` default): each gridline extends into the label gutter
  and ends flush with the labels' outer edge; the label rests on its
  line, and the bottom tick inherits the dark baseline stroke. Applied
  only when the axes has visible y gridlines, so categorical axes are
  untouched; works on either side (e.g. a left latitude axis). Opt out
  with `y_labels="ticks"`, or call `y_labels_on_grid(ax)` manually on
  extra facet panels.
- **Annotations default to 9pt** — `callout`, `highlight_label`,
  `direction_label`, `threshold_arrows` match direct-label size (the
  print spec's 7.5pt reads too small at daily-chart scale).
- **Footnote markers auto-superscript.** `*, †, ‡, §, **, ††, ‡‡, §§`
  render as superscripts anywhere they appear in titles, descriptors,
  source lines, or footnote bodies — write plain text, the renderer
  handles the typography.
- **Frameless legends are default** for both `smart_legend()` and
  `top_legend()`. Boxed legends are opt-in.
- **Source + footnotes use `C_SOURCE`** (`#404040`, the styleguide's
  75% black) — darker than the muted `C_LABEL_MUTED` grey, so
  attribution stays legible while reading as metadata.
- **`finalize` always auto-layouts every margin.** It sizes all four
  `subplots_adjust` margins AND the inter-panel `wspace`/`hspace` from the
  renderer — there is no opt-out and, in the common cases, nothing to set by
  hand. Top/bottom fit the title-stack and the source/footnote band over the
  measured x-tick labels; **left/right are measured from the actual y-axis
  text** (a chart with long left-hung category labels gets a wide left, a plain
  right-axis chart keeps a tight left + a measured right); **`wspace`/`hspace`
  size a grid of panels** (wspace from the inter-column y-label width, hspace
  from the x-tick band + `panel_label` height — pass `finalize(panel_labels=True)`
  for a multi-row faceted layout that adds `panel_label` headings after
  finalize). On a non-Agg backend the side margins and spacing fall back to
  fixed constants. A caller may still `fig.subplots_adjust(...)` *after*
  `finalize` to override a specific value, but the everyday faceted / wide-left
  charts no longer need to. Don't restore `top=` after finalize — it anchors
  the title to its own auto top, so overriding `top` detaches the title; raise
  `y_start` instead to reserve more headroom.
- **Long footnotes word-wrap automatically.** `footnotes(wrap=True)` is the
  default — overflowing notes break to multiple lines that stack above the
  source line, and the chart shifts up to reserve room. No need to hand-
  break with `\n`.
- **Orphan footnote markers warn.** `footnotes(check_anchors=True)` (default)
  raises a `UserWarning` when a note starts with `*` / `†` / `‡` / `§`
  that isn't found in the title, descriptor, axis labels, or any in-chart
  text. Anchor the marker by adding it after a word in the descriptor
  (e.g. `descriptor="Verified contracts* per TVL bucket"`).

### Vertical bar charts (plug-and-play)

For the common "one bar per category" layout, the three-helper combo
collapses ~30 lines of per-script spine/grid/tick boilerplate:

```python
bars = bar_v(ax, labels, values, log=True)               # spines, grid, ticks
bar_value_labels(ax, bars, fmt="{:,}", skip_zero=True)   # value above each bar
bar_sublabels(ax, bars, [f"n={n}" for n in counts])      # denominator below
finalize(ax, title="…", descriptor="…", source="…")
```

`bar_v` highlights the max in `C_RED` by default; pass `highlight_idx=` to
spotlight a specific bar, or `highlight_max=False` to skip. `bar_sublabels`
auto-pads the x-tick labels down so the sublabel band doesn't collide.


## Core design principles

Rules the helpers were built to enforce. When in doubt, satisfy the most.

- **Title does the talking** — state the finding, not the topic. Subtitle
  carries units, geography, time range.
- **Every element earns its place** — if removing it doesn't hurt
  comprehension, remove it.
- **Put information where the reader's eye is already going** — labels next
  to what they label.
- **Direct labelling beats legends.** (See `label_lines`.)
- **Each chart type has its own conventions.** Pick the type first, then
  apply its rules. (See `cycle_for`.)
- **Hue communicates kind; saturation communicates importance.** Same-kind
  differentiation uses saturation or opacity, not a new hue.
- **Brand colour and data colour are separated** — `C_RED` is reserved for
  emphasis, not as the default fill.
- **Four categories is a working ceiling** for thermometer / bubble / pie /
  doughnut. The fix is usually a different chart, not more colours.
- **Don't truncate scales** to fit the comparison.
- **The chart type carries assumptions** — lines imply continuity, bars
  imply discrete categories.
- **Axis scaling is editorial, not neutral.**
- **Double scales need discipline** — align zero lines, match axis colour to
  its line. Otherwise split or index.
- **One chart, one baseline.** If you can't combine cleanly, split.
- **Signal non-zero baselines visually** — `broken_axis()` on line /
  thermometer / scatter. **Never on bar/column** (use a thermometer).
- **Annotations are first-class elements** — callouts, highlight panels,
  event markers have their own typography and geometry.
- **Mobile is a redesign, not a resize.**
- **Negative space is engineered** — the paddings and axis ends are
  deliberate.
- **Sometimes the right answer is less data** — cut or show an average.
- **The chart is responsible to the reader, not to the data.** Misleading,
  confusing, pointless — three failure modes every chart has to pass.

## Headline conventions

The three strings passed to `finalize(title=, descriptor=, source=)` carry
distinct jobs — get them wrong and a technically correct chart still reads
as a draft. The essentials:

- **Title states the finding, not the topic** — "Eastern promise", not
  "GDP growth, 2010-2024". ≤50 characters, no units/dates/geography.
- **Descriptor carries the meat in one breath**: subject, geography/scope,
  period, units — `"United States, CPI, % year on year"`.
- **Footnotes clarify specific words**, anchored by `*`/`†` markers placed
  in the title or descriptor (auto-superscripted).
- **Source names the entity** (`Source:` / `Sources:`); synthetic or
  experimental data cites the generating file. Markdown links survive into
  SVG/PDF output.

Read [references/headline-conventions.md](./references/headline-conventions.md)
before writing the strings for a publishable chart — it carries the worked
examples, footnote categories, hyperlink rules and a before/after table.

## Palette

Nine main colours in `PALETTE`: red, blue, cyan, green, yellow, olive,
purple, gold, grey. Default cycle leads with red. Pull a chart-type-specific
order with `cycle_for("bar" | "bar_stacked" | "line" | "scatter" | "bubble"
| "thermometer")`. `snapshot_palette(n, *, accent=None)` returns a
chronological slate→accent ramp for the "snapshots of the same series over
time" pattern.

Structural greys (all automatic in the theme): `C_SPINE` (zero-baseline
only), `C_GRID`, `C_LABEL`, `C_LABEL_MUTED`, `C_SOURCE` (75% black —
source + footnotes), `C_CI`, `C_BOX_FILL`, `C_BG`, `C_BG_TINT`,
`C_BG_TRANSPARENT`, `C_HIGHLIGHT_PANEL`, `C_HIGHLIGHT_PANEL_RED`.

Style overrides to apply on top:

- **Chronological categories** → light-to-dark tints of one colour, not the
  cycle. Use `snapshot_palette()`.
- **"Other" / "Don't know"** → `C_OTHER` (slate grey).
- **Positive vs. negative** → only differentiate by colour for *meaningful*
  pairs (imports/exports, gain/loss).

## API

### Theme, finalisation, layout

| Function                                                            | Purpose                                                |
|---------------------------------------------------------------------|--------------------------------------------------------|
| `set_theme(bg=None, transparent=False)`                             | Apply theme globally. Call once. White background by default; pass `C_BG_TRANSPARENT` + `transparent=True` for transparent output. |
| `subplots(format="daily", *, height=None, **kwargs)`                | `plt.subplots` at a standard chart width — `"daily"` 4.6in / `"wide"` 7.0in; height is the per-chart choice. |
| `finalize(ax, title, descriptor, source, *, marker="delta", y_labels="on_grid", panel_labels=False, …)` | Title stack (auto-wrapped), optional marker, source line, y-axis right, on-grid y labels. Auto-sizes ALL margins + inter-panel `wspace`/`hspace` from the renderer (left/right from the y-axis text, spacing from a grid); `panel_labels=True` for multi-row facets that add `panel_label` after. Override a specific value with `subplots_adjust` after if ever needed. |
| `panel_label(ax, label)`                                            | Bold sub-heading + dark rule (faceted charts).         |
| `footnotes(fig, *notes, source=None, wrap=True, check_anchors=True)` | Smart-packing footnote strip + optional source line. Auto-superscripts `*, †, ‡, §, **, ††, ‡‡, §§`. Long notes word-wrap to fit the figure (`wrap=True`, default). Warns when a leading marker has no anchor in the title/descriptor (`check_anchors=True`). |
| `y_axis_label(ax, text, *, unit=None)`                              | Horizontal title above the y-axis; `unit=` renders below in muted colour. |
| `x_axis_label(ax, text, *, labelpad=None)`                          | Project-styled `set_xlabel` (`C_SPINE`, 8.5pt); footnote markers superscripted by `finalize`. |
| `year_axis(ax, *, abbreviate=True)`                                 | Date x-axis formatter: first year full, subsequent two-digit. |
| `year_ticks(ax, years, *, inset=True)`                              | Same convention for numeric year axes: full first/century years, two-digit otherwise, inset ends. |
| `x_axis_top(ax)`                                                    | Move the value axis to the top (horizontal-chart convention); call after `finalize()` when a legend row intervenes. |
| `save_chart(__file__)`                                              | Standard save epilogue: `<script>.png` beside the script, tight bbox, 150 dpi, close. |

### Chart helpers

| Function                                                            | Purpose                                                |
|---------------------------------------------------------------------|--------------------------------------------------------|
| `cycle_for(chart_type) → list[str]`                                 | Recommended colour order for a chart type.             |
| `snapshot_palette(n, *, accent=None)`                               | Chronological slate→accent ramp for snapshot lines.    |
| `bar_h(ax, categories, values, *, highlight_max=True)`              | Horizontal bars; max in `C_RED` by default.            |
| `bar_v(ax, categories, values, *, highlight_max=True, log=False, headroom=1.25, y_formatter=None)` | Vertical bars; handles spine/grid/tick boilerplate. `log=True` gives symlog + auto decade ticks. |
| `bar_value_labels(ax, bars, *, fmt="{:,.0f}", formatter=None, fontsize=9)` | Annotate each bar with its value above the bar. Pass `formatter` for currency / mixed units. |
| `bar_sublabels(ax, bars, labels, *, fontsize=8, offset_pt=4)`       | Per-bar secondary text below the baseline (denominators, "n=…"). Auto-pads x-tick labels down to clear the band. |
| `dumbbell(ax, categories, start, end, *, label_start, label_end)`   | Before/after dot-and-line. Defaults red→blue.          |
| `thermometer(ax, categories, values, *, series_labels, dot=True)`   | Tick-and-dot ranked categories. Warns above 4 series.  |
| `threshold_lollipop(ax, categories, values, *, threshold=1.0)`      | Horizontal lollipop with fixed centre + leader lines.  |
| `bump_chart(ax, ranks, *, highlight, aspect=…, max_rank=…)`         | Rank-over-time PCHIP-smoothed lines with white halo at crossings. `max_rank` crops the rank axis so a large backdrop can't compress the story band. |
| `dot_plot(ax, categories, series, *, size=150)`                     | Cleveland dot plot; same-value dots in a row merge into pie-split markers. Returns legend handles. |
| `pie_marker(ax, x, y, wedge_colors, *, size=150)`                   | One marker split into N equal wedges so overlapping points stay visible. |
| `scatter_standard(ax, x, y)`                                        | General-trend scatter, 50% opacity, no stroke.         |
| `scatter_highlight(ax, x, y)`                                       | Outlier / labelled scatter, 100% opacity.              |
| `scatter_category(ax, x, y)`                                        | Bubble dot, 50% fill + 0.3px stroke for overlap.       |
| `trend_line(ax, x, y)`                                              | Dashed 1px trend line.                                 |
| `smoothed_line(ax, x, y, *, color)`                                 | Three-layer scatter + CI band + smoothed line.         |
| `ci_fill(ax, x, lo, hi, *, color=None)`                             | CI band. Salmon by default; pass colour to match line. |

### Annotations

| Function                                                            | Purpose                                                |
|---------------------------------------------------------------------|--------------------------------------------------------|
| `callout(ax, xy, text, *, xytext, arrow=True)`                      | Pale-fill text callout with optional arrow.            |
| `highlight_panel(ax, x_start, x_end, *, label=None)`                | Vertical event-period band.                            |
| `highlight_label(ax, xy, text, *, role="primary")`                  | Single-point label. `"secondary"` = grey/caps/light.   |
| `index_marker(ax, x, *, y=100)`                                     | Red rule + black dot for index charts.                 |
| `broken_axis(ax, *, axis="y", side="auto")`                         | Non-zero-baseline squiggle. Line/scatter/thermometer. `side="auto"` follows the y-label side. |
| `direction_label(ax, text, xy, *, arrow="↑")`                       | One-sided directional cue ("↑ Older husband") in axes-fraction coords. |
| `number_box(ax, xy, n)`                                             | Numbered cross-reference box.                          |
| `threshold_arrows(ax, threshold, *, left_text, right_text)`         | Directional label pair straddling a threshold.         |

### Labels, axes, legends

| Function                                                            | Purpose                                                |
|---------------------------------------------------------------------|--------------------------------------------------------|
| `label_lines(ax, *, stroke=True)`                                   | Direct labels at line ends with collision avoidance; white halo on by default. |
| `inset_tick_labels(ax, *, axis="x")`                                | First tick label `ha="left"`, last `ha="right"`.       |
| `y_labels_on_grid(ax)`                                              | Sit y tick labels on gridlines extended under them (the `finalize()` default; call manually on extra facet panels). |
| `italicize_labels(ax, labels)`                                      | Italicise specific tick labels in place.               |
| `style_labels(ax, *, italic=(), bold=())`                           | Per-label italic/bold preserving tick colour.          |
| `color_axis(ax, side, color, *, spine=True, ticks=True)`            | Colour a spine + ticks + labels to match a series.     |
| `right_axis(ax)`                                                    | Apply right-axis convention to a panel.                |
| `smart_legend(ax)`                                                  | Frameless legend in the emptiest corner.               |
| `top_legend(fig, handles, labels, *, x=0.02)`                       | Frameless top-anchored legend under the title-stack. Call BEFORE `finalize` (auto-`y`) and it reserves its own band — no `subplots_adjust`. |

### Number formatting

| Function                                                            | Purpose                                                |
|---------------------------------------------------------------------|--------------------------------------------------------|
| `format_count(n, *, sig=2)`                                         | Compact count label: 2 s.f. + k/M/B/T unit (`1234 → "1.2k"`, `500 → "500"`). |
| `scale_axis(ax, *, axis="y", by=1000)`                              | Divide an axis's tick labels by a shared magnitude (`20,000 → 20`); name the magnitude in the descriptor. |
| `magnitude_word(by)`                                                | English name for the divisor (`1000 → "thousand"`) — pairs with `scale_axis` for the descriptor unit. |

### Verification

| Function                                                            | Purpose                                                |
|---------------------------------------------------------------------|--------------------------------------------------------|
| `verify_layout(fig, *, tolerance=0.005)`                            | Warn when any text artist (tick labels, titles, legends, footnotes) extends past the figure bounds. Catches the class of bug where `savefig(bbox_inches="tight")` silently expands the saved canvas to fit overflow. Auto-called by `footnotes()`. |

## Workflow

### Build

1. `set_theme()`.
2. (If not the default cycle) `ax.set_prop_cycle(color=cycle_for("…"))`.
3. Plot — `ci_fill` and `highlight_panel` first so they sit behind the data.
4. Annotate — `callout`, `highlight_label`, `index_marker`, `broken_axis`.
5. Direct label — `label_lines(ax)` over `smart_legend(ax)` for line charts.
6. `finalize(ax, title, descriptor, source)` last — auto-sizes margins.
7. Faceted: `finalize()` on the first axes with `title_x` pinned and
   `y_start≈0.075` (reserves the title-stack room) — it auto-sizes the
   inter-panel `wspace`/`hspace` from the panels' y-labels and x-tick band,
   so you usually set NO `subplots_adjust`. Pass `panel_labels=True` for a
   multi-row layout, then call `panel_label()` per axes (they anchor to the
   final positions). Only reach for a post-`finalize` `subplots_adjust` if a
   bespoke layout (hand-placed gutter artists) still needs one.

### Review

**Building the chart is half the job. Reading it back is the other half.**
Every figure should be opened (with the `Read` tool inline, or in a viewer)
and evaluated against the story it was made to tell. Don't ship a draft.

For each rendered figure, answer three questions before moving on:

1. **Does the chart tell its intended story at a glance?**
   If the reader needs body text to know what to look at, the title is
   doing too little. Re-read the title — does it state the *finding*, or
   only the topic? "Eastern promise" tells you what to see; "GDP growth,
   2010–2024" doesn't. Iterate until the title carries the story alone.

2. **Is every element earning its place?**
   Bars at the rounding-noise threshold, legends that duplicate direct
   labels, gridlines that don't anchor anything, sub-labels nobody will
   read, decimal places past the data's precision — cut them. **Less data
   often makes the point sharper.** A six-row table reduced to its three
   meaningful rows is a better chart, not a smaller one. Apply the
   "Every element earns its place" core principle ruthlessly.

3. **Is this the right chart type?**
   Switching costs nothing — the script is ten lines.
   - Six-bucket bar chart trying to show "climbs then dips"? → **line
     chart**.
   - Two-series grouped bars comparing the same metric at two points in
     time? → **dumbbell**.
   - Scatter with 200 points and a single highlight? → **histogram +
     `callout`** (or `scatter_standard` + `scatter_highlight`).
   - Pie / doughnut with more than four slices? → **`bar_h`** ranked.
   - "How does X depend on Y" with strong trend? → **`smoothed_line`**,
     not raw scatter.
   - Long-form ranking change over time? → **`bump_chart`**, not stacked
     bars.

   The chart-type table in `cycle_for()` and the per-type rules in the
   core design principles are the reference.

If any answer is "no" or "kind of", iterate — re-title, cut elements, or
switch chart type. The first render is a draft. `verify_layout()` (auto-
called from `footnotes()`) catches mechanical overflow bugs, but it
cannot tell you whether the chart is *good* — that part is editorial,
not mechanical.

## Examples

Runnable scripts in `examples/`. Each entry leads with the situation that
should make you open it — match your data shape and story shape against the
**Load when** clause, then read the script as the worked reference for that
pattern.

- [`bar_chart.py`](./examples/bar_chart.py) — **Load when:** one value per category, ranked, and the biggest (or one chosen) value is the point. Minimal `bar_h` demo with the default `highlight_max=True`.
- [`dumbbell_chart.py`](./examples/dumbbell_chart.py) — **Load when:** each category has a before and an after value and the change is the message. `dumbbell` plus a right-aligned `top_legend` fed from `ax._dumbbell_handles`.
- [`faceted_chart.py`](./examples/faceted_chart.py) — **Load when:** several series share an axis but overlap so badly that one panel buries the story — split into small multiples. The faceting workflow: `right_axis` + `ci_fill` per axes, `finalize(y_start=0.075)` (auto-sizes the inter-panel `wspace` from the per-panel labels — no manual `subplots_adjust`), then `panel_label` per axes (anchored to the final positions).
- [`faceted_top_legend.py`](./examples/faceted_top_legend.py) — **Load when:** faceted panels share a colour key that must sit above the row of panels, hands-off. `top_legend` is called BEFORE `finalize`; `finalize` measures it, reserves a band between the two-line descriptor and the panels, and re-anchors it to the final axes top — no `subplots_adjust`, no hand-tuned `y=` / `y_start` padding.
- [`scatter_chart.py`](./examples/scatter_chart.py) — **Load when:** a relationship cloud where a few named outliers carry the story. `scatter_standard` + `scatter_highlight` two-layer treatment, `trend_line`, `callout` on the outliers.
- [`thermometer_chart.py`](./examples/thermometer_chart.py) — **Load when:** 2–4 series compared across the same ranked categories, where grouped bars would clutter. `thermometer(dot=False)` three-series variant, x-axis on top, frameless `top_legend`.
- [`index_chart.py`](./examples/index_chart.py) — **Load when:** series of different magnitudes must be compared as growth since a common moment, with an event window to flag. `index_marker` + `highlight_panel` band + secondary `highlight_label` + `broken_axis` + `label_lines`.
- [`line_chart.py`](./examples/line_chart.py) — **Load when:** noisy point estimates over time with uncertainty — the reader should see trend and range, not individual points. `smoothed_line` (scatter + CI band + trend), custom `_LineBandHandler` legend, `year_axis(set_locator=False)`.
- [`bump_chart.py`](./examples/bump_chart.py) — **Load when:** the story is rank changes over time in a large field, with only a few entities highlighted against a faded backdrop. `bump_chart` with `highlight=`, `colors=`, `right_labels=True`, `x_labels_top=True`, `max_rank` cropping; real data via `_data.py`.
- [`corbyn.py`](./examples/corbyn.py) — **Load when:** ranked bars where one dominant value tempts you to truncate the scale (don't — show it whole), or specific row labels need bold/italic emphasis. `bar_h` + `style_labels(italic=, bold=)`.
- [`dogs.py`](./examples/dogs.py) — **Load when:** two different units genuinely must share one panel and the twin axes have to be honest — both ranges sized so 1% of the midpoint spans equal distance. `color_axis(spine=False, ticks=False)`, manual series titles via `render_text_with_superscripts`, `footnotes(source=)` packing two notes.
- [`brexit.py`](./examples/brexit.py) — **Load when:** irregular poll/survey readings over time — connecting the dots would look erratic, so show points plus a smoothed trend on a truncated axis. `scatter_standard` + Savitzky-Golay smoothing, manual year ticks + `year_axis(set_locator=False)`, `inset_tick_labels`, `broken_axis(axis="both")`.
- [`us_trade.py`](./examples/us_trade.py) — **Load when:** two related time series with incompatible units or baselines tempt you toward a double axis — split into stacked `sharex` panels instead. `panel_label`, `right_axis`, `inset_tick_labels`, source via `footnotes(source=)`.
- [`pensions.py`](./examples/pensions.py) — **Load when:** a scatter where labelled points need emphasis but a second colour would falsely read as a category — use opacity within one hue. Same-hue dots with opacity-for-emphasis, italic average label via `FontProperties`, `y_axis_label(unit=)`.
- [`eu_balance.py`](./examples/eu_balance.py) — **Load when:** a stacked composition has too many members to colour — keep the named few plus an "Others" bucket, and show two measures as side-by-side panels. Positive+negative stacked bars per year, shared `top_legend`, `right_axis`.
- [`affordability_chart.py`](./examples/affordability_chart.py) — **Load when:** categories are measured against a meaningful threshold (above = fine, below = not) and span a wide value range. `threshold_lollipop(threshold=1.0)` on a log x-axis + `threshold_arrows` straddling the threshold + two-note `footnotes`.
- [`age_gap_chart.py`](./examples/age_gap_chart.py) — **Load when:** the same curve measured at several historical moments, and the story is how its shape shifted — earlier years fade, the present pops. Chronological lines via `snapshot_palette(4)`, in-chart series labels, `broken_axis(side="right")`, right-anchored `footnotes(y=, x=)`.

### Daily-chart replicas

Faithful replicas of Economist daily charts (references via
`examples/fetch_refs.py`, comparisons via `examples/build_comparisons.py`).
Beyond fidelity checks, these double as the worked examples for less-common
data and story shapes the core helpers don't cover directly.

- [`australia_heat.py`](./examples/australia_heat.py) — **Load when:** a time series of signed deviations around zero where direction is the story. Diverging annual `bar_v` bars, positive red / negative blue on a zero baseline.
- [`malaria.py`](./examples/malaria.py) — **Load when:** observed history fans out into multiple scenario paths and the reader must not confuse projection with fact. Solid history line splitting into three dashed forecasts inside a `highlight_panel` FORECAST band.
- [`co2_emissions.py`](./examples/co2_emissions.py) — **Load when:** each category has two multiplicative dimensions (rate × size = total) and the totals matter as much as the rates. Variable-width Mekko bars with on-bar totals + dashed global-average rule.
- [`christianity.py`](./examples/christianity.py) — **Load when:** many categories each have exactly two time points and the trajectories — who's rising, who's flat — are the message. Two-point slope chart with endpoint dots, stacked value labels, one accent line.
- [`graduate_pay.py`](./examples/graduate_pay.py) — **Load when:** a dense dot cloud with a strong nonlinear trend, where the x variable reads better reversed ("more selective →"). `scatter_standard` cloud, reversed x-axis, solid trend curve.
- [`generational_politics.py`](./examples/generational_politics.py) — **Load when:** cohort or panel-survey waves — several groups tracked over irregular waves, with gaps in coverage and an overall average. PCHIP-smoothed lines, gap segments in lighter tint, dotted average, end-of-line labels.
- [`uber_tips.py`](./examples/uber_tips.py) — **Load when:** paired group values per category, expressed relative to an explicit reference point that needs marking. Vertical dumbbell/lollipop pairs with a ringed reference marker and leader line.
- [`us_refugees.py`](./examples/us_refugees.py) — **Load when:** an actual annual quantity tracked against an administrative limit — the gap between allowed and delivered is the story. Neutral grey bars + stepped red policy-cap line (real published data).
- [`polluted_cities.py`](./examples/polluted_cities.py) — **Load when:** the ranking itself is the chart — a top-N list where membership patterns matter more than precise values, and >~10 rows need splitting into blocks. Two-block ranked table with colour-graded value chips and selective row bands.
- [`arctic_warming.py`](./examples/arctic_warming.py) — **Load when:** a value profiled across an ordered spatial dimension (latitude, depth, distance), with a per-point range across sources. Dot-line over translucent range bars, x-axis on top, banded regions of interest.
- [`trump_sanctions.py`](./examples/trump_sanctions.py) — **Load when:** annual-count bars where eras or regimes are the comparison frame. `bar_v` time series on tinted presidential-era `highlight_panel` bands, `inset_tick_labels`.
- [`populist_votes.py`](./examples/populist_votes.py) — **Load when:** a two-part composition over a long period where both the total and the mix change — the flip is the story. Two-series stacked bars across 40 years, `top_legend`.
- [`plastic_bottles.py`](./examples/plastic_bottles.py) — **Load when:** composition shares across a few snapshots, and the reader also needs geographic orientation for an obscure location. Stacked 100% bars + orthographic locator-globe inset under the legend.
- [`alcohol_drinkers.py`](./examples/alcohol_drinkers.py) — **Load when:** the same group breakdown applied across several different denominators (population vs revenue vs consumption) — the mismatch is the point. Three stacked 100% horizontal bars under a shared pictogram legend strip.
- [`language_speed.py`](./examples/language_speed.py) — **Load when:** comparing full distributions per category, not summary statistics — especially when a second panel shows the distributions collapsing together. Two-panel ridgeline densities (plain `fill_between`), `direction_label` reading cues, `panel_label`.
- [`london_roads.py`](./examples/london_roads.py) — **Load when:** one metric over two cyclic dimensions (day × hour) where the pattern lives in the grid, not in any single series. Heatmap with a discrete sampled colour scale and hand-built legend.
- [`elderly_screens.py`](./examples/elderly_screens.py) — **Load when:** composition totals over time compared between two groups — both the level and the mix matter, so neither lines nor 100% bars suffice. Two stacked-area panels on a shared scale, `right_axis`.
- [`wework.py`](./examples/wework.py) — **Load when:** comparing two entities across several metrics of wildly different scales — per-panel normalisation so every comparison spans its band. Per-metric banded panels with paired bars, per-panel x-scales, value labels inside or beyond the bar tip.
- [`millennial_parents.py`](./examples/millennial_parents.py) — **Load when:** chronological snapshot lines (as `age_gap_chart.py`) but specific snapshots must pin to exact colours from a reference. `snapshot_palette` with per-line colour overrides.
- [`bad_bunny.py`](./examples/bad_bunny.py) — **Load when:** response shares for a single survey question — ranked `bar_h` (as `bar_chart.py`) but with the survey convention of the x-axis on top. `bar_h` + `footnotes`.
- [`spending_convergence.py`](./examples/spending_convergence.py) — **Load when:** two lines converging or crossing, where the meaningful window forces a truncated y-axis that must be signalled, not hidden. Crossing line pair with the `broken_axis` squiggle beside the right tick column.
- [`gold_rally.py`](./examples/gold_rally.py) — **Load when:** comparing returns over a sub-year window — rebased to 100 (as `index_chart.py`) but with dense daily data and month-resolution ticks. `index_marker` + month-letter ticks, direct line labels.
- [`nuclear_warheads.py`](./examples/nuclear_warheads.py) — **Load when:** ranked totals per entity with an internal status breakdown, plus a forecast for one entity that must read as projection. Stacked `barh` inventories, top axis + `top_legend`, dashed forecast box.

---
> Source: [DJRHails/graphs](https://github.com/DJRHails/graphs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
