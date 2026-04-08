## jupyter-estructural-v2

> This document provides a comprehensive guide for Gemini, the AI assistant, to effectively contribute to the AGP (Análisis General de Postaciones) project. It synthesizes all project-specific rules, architectural patterns, and development guidelines.

# Gemini Development Guide for AGP Project

This document provides a comprehensive guide for Gemini, the AI assistant, to effectively contribute to the AGP (Análisis General de Postaciones) project. It synthesizes all project-specific rules, architectural patterns, and development guidelines.

## 1. Project Overview

**Purpose:** AGP is a specialized engineering application for designing and analyzing electrical transmission line structures, ensuring compliance with AEA (Asociación Electrotécnica Argentina) standards.

**Key Features:**
- Structural and mechanical design of power line towers.
- Cable tensioning, wind load, and pole selection calculations.
- Interactive Dash/Plotly dashboard for visualization and analysis.
- 3D rendering of structures and load distributions.
- Management of structure families and cost analysis.

**Target Users:** Electrical and structural engineers involved in transmission line design.

## 2. Core Architectural Principles

- **MVC Pattern:**
    - **Models (`/models`):** `AppState` manages data and persistence.
    - **Views (`/views`, `/components`):** Dash components render the UI.
    - **Controllers (`/controllers`):** Handle user interactions, business logic, and callbacks.
- **Separation of Concerns:** UI is decoupled from the core calculation engines.
- **Configuration-Driven:** Project behavior is controlled via `config/app_config.py` and structure-specific JSON files.
- **File-Based Persistence:**
    - Structure definitions (`.estructura.json`), calculation results (`.calculo*.json`), and cache are stored in the `/data` directory.
    - **CRITICAL:** Always sanitize filenames by replacing spaces with underscores (e.g., `nombre.replace(' ', '_')`) in both save and load operations.
- **Caching System (`/data/cache`):**
    - `CalculoCache` class manages all persistence.
    - Results are cached to prevent redundant, expensive calculations.
    - Filenames are based on structure name and calculation type (e.g., `MyProject.calculoCMC.json`).
    - An MD5 hash of structure parameters determines cache validity.

## 3. Critical Development Rules & Protocols

### **Rule 1: Adherence to Existing Code**
- **ALWAYS** review existing code in similar modules (`controllers`, `components`, `utils`) before implementing new features.
- **REUSE** existing patterns for callbacks, UI components, and data handling to maintain consistency. Do not invent new patterns.

### **Rule 2: No Mock/Placeholder Code**
- **NEVER** use placeholder implementations, dummy data, or hardcoded values that should be calculated.
- If data is missing, raise a `ValueError` or return `None`. Do not invent fallback data.
- If a feature cannot be fully implemented, clearly mark it as pending and notify the user. Do not create fake code that appears to work.

### **Rule 3: Python Script Execution**
- To run any Python script, **ALWAYS** use the following command structure to ensure the correct virtual environment is used:
  ```bash
  cd C:\Users\gpesoa\Downloads\portablepy\venvo\Scripts && python.exe [FULL_PATH_TO_SCRIPT]
  ```
- **Example:**
  ```bash
  cd C:\Users\gpesoa\Downloads\portablepy\venvo\Scripts && python.exe C:\Users\gpesoa\MobiDrive\jupyter_estructural_v2\app.py
  ```

### **Rule 4: Application Execution (No Hot Reload)**
- The application must be run with `debug=False`. Hot reload is **DISABLED** as it interferes with the application's state.
- The standard workflow is:
  1. Make code changes.
  2. Stop the application (Ctrl+C).
  3. Relaunch with `python app.py`.
  4. Manually refresh the browser.

### **Rule 5: QA & Testing Protocol**
- **NEVER** mark an issue as "✅ RESUELTO" (Resolved). This is the user's exclusive responsibility.
- After implementing a fix, the state must **ALWAYS** be marked as "🔧 TESTING PENDIENTE" (Pending Testing).
- The AI's role is to implement fixes, not to verify them.

### **Rule 6: Documentation and File Organization**
- All new `.md` documentation files must be placed in the `/docs` folder.
- Use emojis sparingly. Only colored circles/squares are permitted in code.

## 4. Technology Stack & Key Patterns

- **Language:** Python 3.x
- **Framework:** Dash, Dash-Bootstrap-Components
- **Data:** Pandas, NumPy
- **Visualization:** Plotly, Matplotlib
- **Persistence:** JSON, Openpyxl

### **Dash Callback Best Practices**
1.  **Function-Based Registration:** **NEVER** use standalone `@callback` decorators. All callbacks for a controller must be defined within a `register_callbacks(app)` function. This prevents registration conflicts.
2.  **State vs. Input:** Use `State` for data that is needed but should not trigger the callback. Use `Input` for events that should trigger the callback.
3.  **`prevent_initial_call=True`:** Use this in callbacks to prevent execution on page load. However, always add explicit checks (e.g., `if n_clicks is None:`) as this is not a foolproof guarantee.
4.  **`allow_duplicate=True`:** Use this on `Output`s that are updated by more than one callback (e.g., toast notifications, dynamic tables).
5.  **Stale State:** **CRITICAL:** Dash `State` can become stale. For file-backed data, **ALWAYS** reload the data from the source file (e.g., `actual.estructura.json`) at the beginning of a critical callback instead of trusting the value from `State`.
6.  **Unique IDs:** When creating features similar to existing ones (e.g., a new modal system), use unique ID prefixes for all components and callbacks to avoid `DuplicateCallback` errors.

### **Plotly Graph Handling**
- **CRITICAL:** When saving a Plotly figure to cache, **ALWAYS** save it in **TWO formats**:
    1.  **PNG (`fig.write_image()`):** For static exports.
    2.  **JSON (`fig.write_json()`):** **Required** for the `dcc.Graph` component to render the figure interactively. If the JSON is missing, the graph will not appear when loaded from cache.
- **Capturing Figures:** When a function returns a Plotly figure object, capture the return value directly. Do **NOT** use `plt.gcf()`, as that will capture a Matplotlib figure, leading to errors.

### **DataFrame Handling**
- To preserve exact formatting (column widths, wrapping), serialize DataFrames to JSON using `df.to_json(orient='split')` and load them using `pd.read_json(json_str, orient='split')`.

## 5. Common Troubleshooting Guide

- **`TypeError: Unexpected keyword argument`:** A Dash component received a property it doesn't support (e.g., an internal `'tipo'` field). Filter out internal fields from the dictionary before passing it to the component.
- **Plotly Graphs Not Appearing from Cache:** The figure's JSON file is missing. Ensure `fig.write_json()` is called during the save-to-cache operation.
- **Callbacks Firing Unexpectedly:** Use `dash.callback_context` to debug what triggered the callback. Add explicit `if n_clicks is None:` checks.
- **`DuplicateCallback` Error:** Multiple callbacks are targeting the same `Output` without `allow_duplicate=True`, or component IDs are not unique across the application.
- **Component Lists in Dash:** When building a list of components, use `list.append(component)` for each item. Avoid using `list.extend([title, component])` with complex components, as it can cause rendering issues.
- **Data Contamination:** When creating a new structure from a template, ensure fields like `nodos_editados` are reset to an empty list (`[]`) to avoid carrying over data from a previous structure.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/agerpk)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/agerpk)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
