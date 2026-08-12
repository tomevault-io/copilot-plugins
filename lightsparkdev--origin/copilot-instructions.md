## charts

> Chart component color and usage conventions

# Chart Color Conventions

## Available Components

| Component | Use case |
|---|---|
| `Chart.Line` | Line chart, area chart (via `fill` prop), sparkline-like |
| `Chart.Sparkline` | Compact inline chart — no axes, no interaction |
| `Chart.StackedArea` | Stacked cumulative area bands |
| `Chart.Bar` | Grouped, stacked, or horizontal bar chart |
| `Chart.Pie` | Donut chart with legend sidebar |
| `Chart.Composed` | Mixed bar + line with dual Y-axes |
| `Chart.Gauge` | Arc gauge with thresholds and marker |
| `Chart.BarList` | Horizontal bars with labels — supports rank numbers, change indicators, secondary values |
| `Chart.Uptime` | Binary status timeline (up/down) |
| `Chart.Live` | Canvas streaming chart with `requestAnimationFrame` |
| `Chart.Scatter` | XY scatter plot — multi-series, nearest-point tooltip |
| `Chart.Split` | Segmented distribution bar — parts of a whole |
| `Chart.Sankey` | Flow diagram — multi-path node/link with contextual hover filtering |
| `Chart.Funnel` | Conversion funnel with tapered stages and drop-off rates |
| `Chart.Waterfall` | Waterfall chart — running total with increases, decreases, totals |

## Color Strategy

### Single series

Use `color` prop or `dataKey` — the component defaults to `var(--stroke-primary)`.

```tsx
<Chart.Line data={data} dataKey="value" color="var(--color-blue-600)" />
```

### Multi-series: shade ramps (preferred)

For stacked or grouped data where the series represent parts of a whole, use shades of one hue from the primitive color scale. This creates visual cohesion — the bands read as layers, not competing categories.

```tsx
// 3-series shade ramp
series={[
  { key: 'a', label: 'Primary', color: 'var(--color-blue-700)' },
  { key: 'b', label: 'Secondary', color: 'var(--color-blue-400)' },
  { key: 'c', label: 'Tertiary', color: 'var(--color-blue-200)' },
]}
```

Recommended shade stops by series count:

- **2 series**: 700, 300
- **3 series**: 700, 400, 200
- **4 series**: 700, 500, 300, 100
- **5 series**: 800, 600, 400, 200, 050

### Multi-series: distinct hues

When the series represent fundamentally different categories (e.g., incoming vs outgoing), use distinct semantic colors:

```tsx
series={[
  { key: 'incoming', label: 'Incoming', color: 'var(--surface-blue-strong)' },
  { key: 'outgoing', label: 'Outgoing', color: 'var(--surface-purple-strong)' },
]}
```

### Composed charts (bar + line)

Use a lighter shade for bars (they have large surface area) and full-strength or `--text-primary` for lines (thin, needs contrast):

```tsx
series={[
  { key: 'revenue', type: 'bar', color: 'var(--color-blue-300)' },
  { key: 'rate', type: 'line', axis: 'right', color: 'var(--text-primary)' },
]}
```

## Color Scale Reference

Every hue has 12 stops: `050, 100, 200, 300, 400, 500, 600, 700, 800, 900, 950`.

Available hues: `blue`, `purple`, `green`, `pink`, `red`, `yellow`.

Pattern: `var(--color-{hue}-{stop})`

## Fallback Palette

When no `color` is set on a series, the component auto-assigns from `SERIES_COLORS`:

1. `var(--stroke-primary)` (near-black)
2. `var(--text-secondary)` (gray)
3. `var(--surface-blue-strong)`
4. `var(--surface-purple-strong)`
5. `var(--surface-green-strong)`
6. `var(--surface-pink-strong)`

This palette is designed for distinct-hue multi-series. For stacked/grouped charts, always set explicit shade ramp colors.

## Do / Do Not

- **Do** use shade ramps for stacked and grouped charts
- **Do** use `var(--color-*)` primitive tokens for chart series colors
- **Do** use lighter shades for large-area fills (bars, area bands) and stronger shades for thin elements (lines, dots)
- **Do not** hardcode hex colors — use tokens
- **Do not** rely on the fallback palette for stacked charts — the distinct hues look incohesive when stacked
- **Do not** use semantic surface tokens (`--surface-blue-strong`) when you need shade control — use the primitive scale instead

## Sankey vs Funnel

Use **Funnel** when the flow is strictly sequential — every user passes through every stage in order and you care about drop-off rates between stages.

Use **Sankey** when the flow branches — users take different paths to different outcomes, and you need to show how volume splits and merges across multiple routes.

| | Funnel | Sankey |
|---|---|---|
| Data shape | Linear sequence (A → B → C) | Directed graph (A → B, A → C, B → D) |
| Key metric | Conversion rate between stages | Volume per path |
| Best for | Single-path conversion funnels | Multi-path routing, budget allocation, attribution |

## Interaction Contract

Each chart type exposes a click handler matching its data model:

| Component | Handler |
|---|---|
| `Chart.Line`, `Chart.Bar`, `Chart.Composed`, `Chart.StackedArea`, `Chart.Pie` | `onClickDatum(index: number, datum: Record<string, unknown>) => void` |
| `Chart.Split` | `onClickDatum(segment: SplitSegment, index: number) => void` |
| `Chart.Scatter` | `onClickDatum(seriesKey: string, point: ScatterPoint, index: number) => void` |
| `Chart.BarList` | `onClickDatum(item: BarListItem, index: number) => void` |
| `Chart.Funnel` | `onClickDatum(index: number, stage: FunnelStage) => void` |
| `Chart.Waterfall` | `onClickDatum(index: number, segment: WaterfallSegment) => void` |
| `Chart.Sankey` | `onClickNode(node: LayoutNode) => void` / `onClickLink(link: LayoutLink) => void` |

---
> Source: [lightsparkdev/origin](https://github.com/lightsparkdev/origin) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
