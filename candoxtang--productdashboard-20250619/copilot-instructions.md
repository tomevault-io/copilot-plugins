## productdashboard-20250619

> The application uses a decoupled, event-driven architecture to manage data loading and updates. Understanding this flow is critical for debugging and adding new features.

# Application Data Flow

The application uses a decoupled, event-driven architecture to manage data loading and updates. Understanding this flow is critical for debugging and adding new features.

## 1. Initial Data Generation (`dataReady` event)

*   **Source**: `[js/data-source.js](mdc:js/data-source.js)`
*   **Trigger**: The `DOMContentLoaded` event.
*   **Process**:
    1.  The `initializeDataSource` function in `[js/data-source.js](mdc:js/data-source.js)` begins generating all the necessary data for the dashboards.
    2.  This data is compiled into a single global object, `window.appData`.
    3.  Once generation is complete, it dispatches a custom event: `document.dispatchEvent(new CustomEvent('dataReady', ...))`.
*   **Consumers**:
    *   `[js/main.js](mdc:js/main.js)` listens for this event to know that it's safe to initialize the dashboards.
    *   All individual dashboard modules (e.g., `[js/material-analysis.js](mdc:js/material-analysis.js)`, `[js/quality-dashboard.js](mdc:js/quality-dashboard.js)`) listen for `dataReady`. Upon receiving it, they set an internal flag (e.g., `isDataReady = true`) and perform their initial chart rendering.

## 2. Global Filter Updates (`globalFiltersChanged` event)

*   **Source**: `[js/global-filters.js](mdc:js/global-filters.js)`
*   **Trigger**: User interaction with any of the filter elements in the global filter bar (e.g., selecting a Fab, Product, or changing the aggregation level).
*   **Process**:
    1.  The `[js/global-filters.js](mdc:js/global-filters.js)` script updates its internal state.
    2.  It then dispatches a custom event: `document.dispatchEvent(new CustomEvent('globalFiltersChanged', ...))`.
*   **Consumers**:
    *   Each dashboard module has an event listener for `globalFiltersChanged`.
    *   When the event is received, each module calls its main update function (e.g., `applyFiltersAndRedraw()` in `material-analysis.js`). This function re-filters the data from `window.appData` based on the new filter state and updates its charts.

This two-event system ensures that charts are only rendered when data is available and are efficiently updated when the user context changes.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/candoxtang)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/candoxtang)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
