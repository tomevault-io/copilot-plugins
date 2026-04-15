## data-analisys

> This project is a Wails-based application that bridges a Go backend (performing heavy mathematical analysis) with a Chart.js-powered frontend.

# Gemini Project Documentation: Go-Chart.js Adapter & Render Pipeline

This project is a Wails-based application that bridges a Go backend (performing heavy mathematical analysis) with a Chart.js-powered frontend.

## 🏗 Project Structure

- **`/charting`**: Core Go models for the visualization adapter.
  - `chart.go`: Defines `Chart` and `DataPoint`.
  - `dataset.go`: Defines the `Dataset` interface and concrete types:
    - `GridDataset`: For Scatter, Line, and Bubble charts (uses `DataPoint`).
    - `CategoricalDataset`: For Bar and Pie charts (uses `[]any` for values).
    - `HeatmapDataset`: For Heatmaps (uses `HeatmapPoint`).
  - `render.go`: Defines `RenderRequest` and `RenderResponse`.
- **`/labs`**: Scientific implementations.
  - Each sub-package (e.g., `cluster`, `stats`) contains a `RenderFunc` that maps raw data to the `charting` models.
- **`/frontend/src`**:
  - `chart-render.ts`: The "Brain" of the visualization. Handles canvas creation and Chart.js instantiation.
  - `static-config.ts`: Global Chart.js defaults (colors, fonts, zoom plugins).
  - `events.ts` & `fetch.ts`: Manages the asynchronous bridge between Wails events and the UI.
- **`/uncsv`**: Custom high-performance CSV decoder for `DataPoint` structures.

## 🔄 The Render Flow

The application uses an **Asynchronous Event-Driven Pipeline** to keep the UI responsive during complex calculations.

1.  **Interaction**: User modifies a `MutableField` (slider/select) or clicks **Rerender**.
2.  **JS Request**: `fetchChartData()` gathers all current UI variables into a `RenderRequest`.
3.  **Go Invocation**: `App.Render(req)` is called.
    - It immediately returns to the JS side to free the UI thread.
    - A goroutine is spawned to execute the scientific logic.
4.  **Backend Execution**: The `LabProvider` locates the specific `RenderFunc`.
    - Data is fetched (usually from `/data/*.csv`).
    - Math is performed (e.g., K-Means, Silhouette Scores).
    - A `charting.Chart` object is populated with datasets.
5.  **Event Emit**: Once finished, the Go backend emits a `renderComplete` event via Wails Runtime.
6.  **JS Receipt**: `EventsOn("renderComplete")` in `main.ts` receives the `RenderResponse`.
7.  **Draw**: Based on the `chart.type`, the frontend calls either `renderChartInto` or `renderMultiChart`.

## 📊 Data Display Modes

### 1. Standard Mode (`type: "line" | "bar" | "scatter" ...`)
Renders a single Chart.js instance into the main container.
- **Labels**: Uses global `chart.labels` for the X-axis.
- **Datasets**: All datasets in `chart.datasets` are rendered on the same axes.

### 2. Multi-Chart Mode (`type: "multi-bar" | "multi-scatter" ...`)
Designed for comparisons (e.g., Silhouette plots where each cluster needs its own scale).
- **Logic**: The frontend strips the `multi-` prefix and creates a **Grid Layout**.
- **Synthetic Charts**: For every dataset in the original chart, a "Synthetic Chart" is created.
- **Isolation**: Each cluster/dataset gets its own individual canvas and coordinate system.

## 🛠 Adapter Safety & Best Practices

- **Typed Datasets**: Always instantiate the correct dataset type for your chart:
  - Use `GridDataset` when you have `{x, y}` coordinates.
  - Use `CategoricalDataset` when you have indexed values corresponding to `chart.Labels`.
  - Use `HeatmapDataset` for 3D data visualizations.
- **Labels & length**: Always use `Array.isArray(labels) && labels.length > 0` before processing. The frontend is defensive against `null` labels from the backend.
- **Datalabels Plugin**: 
  - For **Scatter**: Uses `DataLabels` for per-point tooltips/labels.
  - For **Pie/Doughnut**: Automatically calculates and displays percentages.
  - For **Bar**: Usually disabled globally to prevent clutter, unless `DataLabels` are explicitly provided.
- **Colors**: Use `charting.ToColor("#hex")` to ensure color strings are wrapped in the `Color` type.
- **Responsive Height**: All charts use `maintainAspectRatio: false`. The parent container must have a `min-height` (defined in `style.css` as 500px for single and 400px for multi-wrappers) to prevent vertical squishing.
- **Coordinate Systems**: 
  - Use `charting.LinearAxis` for Scatter/Bubble.
  - Use `charting.CategoryAxis` for Bar/Line with discrete labels.

## 🧬 Architecture & Data Integrity

### 1. Handling Missing Data (The `null` pattern)
- **`[]any` over `[]float64`**: The `CategoricalDataset.Data` field uses `[]any` to allow `nil` values.
- **Visual Integrity**: Always use `nil` for indices where data is unavailable (e.g., early forecasting steps). This prevents Chart.js from jumping to `0`, which skews the Y-axis.
- **Conversion**: Use `charting.ToAnySlice(data []float64)` to safely convert standard slices for the adapter.

### 2. Variable Registration & Metadata
- **Static Declaration**: All `MutableField`s (for both Charts and specific Datasets) must be declared and assigned to the `Chart` template during package initialization (`var` or `init`).
- **Discovery**: The `NewLabConfig` constructor scans these templates to generate metadata. Fields added dynamically during `Render` will not be visible in the UI sidebars/controls.
- **Graph-Level Stats**: Use the `GraphVariables` slice on a `ChartDataset` to display read-only statistics (using `ControlNoControl`) specific to that dataset.

### 3. State Management
- **Surgical Updates**: Always use `copyChart.UpdateDataForDataset(id, data)` or `copyChart.UpdatePointsForDataset(id, x, y)` instead of direct field assignment to ensure internal consistency.
- **Immutable Templates**: Never modify the global `Chart` template. Use `charting.CopyChart(Template)` at the start of every `RenderFunc`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/yur4uwe)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/yur4uwe)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
