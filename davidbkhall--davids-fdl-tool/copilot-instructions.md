## davids-fdl-tool

> This is a **macOS-native** desktop application. All code, UI, and platform assumptions target macOS exclusively (macOS 14 Sonoma+). Do not introduce cross-platform abstractions, UIKit patterns, or iOS/Linux/Windows compatibility shims.

# FDL Tool — Cursor Rules

## Platform

This is a **macOS-native** desktop application. All code, UI, and platform assumptions target macOS exclusively (macOS 14 Sonoma+). Do not introduce cross-platform abstractions, UIKit patterns, or iOS/Linux/Windows compatibility shims.

## Tech Stack

### Frontend — Swift / SwiftUI
- **Language:** Swift 5.9+
- **UI Framework:** SwiftUI (macOS-native, `NavigationSplitView`, sheets, inspectors)
- **Build System:** Swift Package Manager (`FDLTool/Package.swift`)
- **Minimum Deployment Target:** macOS 14
- **Dependencies:**
  - SQLite.swift (0.15.0+) for local database access

### Backend — Python
- **Language:** Python 3.10+
- **IPC:** JSON-RPC 2.0 over stdin/stdout (newline-delimited JSON, one object per line)
- **Key Libraries:** fdl (ASC FDL core bindings), Pillow, svgwrite, cairosvg
- **System Dependencies:** ffmpeg/ffprobe

### ASC FDL Core Reference Library
This project depends on the **ASC Framing Decision List reference implementation** maintained at <https://github.com/ascmitc/fdl/tree/dev>. The `fdl` Python package provides ctypes bindings over the native C library (`libfdl_core`). When working with FDL parsing, validation, geometry, canvas resolution, or template logic, defer to the behavior and API defined by this reference library. Do not reimplement FDL spec logic that already exists in the `fdl` package.

### FDL Schema — v2.0.1
All FDL documents in this project use the **ASC FDL v2.0.1 schema** (`schema/v2.0.1/ascfdl.schema.json`). Key field names:
- **Top level:** `uuid`, `version` (`{major, minor}`), `fdl_creator`, `default_framing_intent`, `framing_intents`, `contexts`, `canvas_templates`
- **Canvas:** `id`, `source_canvas_id`, `dimensions`, `effective_dimensions`, `effective_anchor_point`, `photosite_dimensions`, `anamorphic_squeeze`, `framing_decisions`
- **Framing Decision:** `id` (format: `canvasId-intentId`), `framing_intent_id`, `dimensions`, `anchor_point`, `protection_dimensions`, `protection_anchor_point`
- **Do NOT** use legacy field names (`header`, `fdl_contexts`, `canvas_uuid`, `fd_uuid`, `anchor`, `framing_intent`, `effective_anchor`, `protection_anchor`). The `_v1_to_v2()` converter in `fdl_convert.py` exists solely for importing old files.

## Architecture

### MVVM (Swift Frontend)
- Each tool/feature has a **View** (SwiftUI) and a **ViewModel** (`ObservableObject`).
- **AppState** is a shared `@MainActor ObservableObject` holding cross-tool state: the current project, Python bridge status, and service stores (`LibraryStore`, `CameraDBStore`).
- **Models** are `Codable` structs in `Models/`.
- **Services** handle backend communication and data persistence in `Services/`.
- **Navigation** uses `NavigationSplitView` with a sidebar tool selector.

### Python Backend (JSON-RPC Server)
- Handlers live in `fdl_backend/handlers/`, one module per domain (fdl_ops, template_ops, chart_gen, image_ops, clip_id).
- Each handler function takes a `dict` of params and returns a `dict` result.
- The server dispatches methods like `fdl.validate` → `fdl_ops.validate(params)`.
- The `PythonBridge` Swift actor manages the Python subprocess lifecycle.

### IPC Contract
- Requests: `{"jsonrpc":"2.0","id":N,"method":"domain.action","params":{...}}`
- Responses: `{"jsonrpc":"2.0","id":N,"result":{...}}` or `{"jsonrpc":"2.0","id":N,"error":{"code":N,"message":"..."}}`
- The Python process signals readiness by writing `"FDL backend server ready\n"` to stderr.

## Data Storage
- **SQLite** database at `~/Library/Application Support/FDLTool/fdltool.db`
- **FDL files** at `~/Library/Application Support/FDLTool/projects/{project_id}/{entry_id}.fdl.json`
- **Canvas templates** at `~/Library/Application Support/FDLTool/templates/{template_id}.json`

## Project Layout
```
FDLTool/                          Swift Package (macOS app)
  Sources/FDLTool/
    App/                          FDLToolApp, AppState
    Navigation/                   SidebarView, NavigationModel
    Library/                      FDL library management views/VMs
    Viewer/                       FDL viewer, frameline overlay, tree view
    ChartGenerator/               Framing chart generation views/VMs
    CanvasTemplates/              Canvas template editor/list views/VMs
    ClipID/                       Clip identification views/VMs
    CameraDB/                     Camera database browser views
    Shared/                       Reusable UI components
    Models/                       Codable data models
    Services/                     PythonBridge, LibraryStore, CameraDBStore, FFProbeService
  Tests/FDLToolTests/

python_backend/                   Python JSON-RPC service
  fdl_backend/
    server.py                     JSON-RPC stdin/stdout server loop
    handlers/                     Domain handler modules
    camera_db/                    Camera database sync/models
    utils/                        Shared utilities (SVG renderer, ffprobe)
  tests/

resources/                        Bundled data (camera DB, FDL schemas, samples)
scripts/                          Setup and build scripts
```

## Coding Preferences

### Swift
- Use Swift concurrency (`async`/`await`, actors) for all asynchronous work. Avoid Combine for new code unless extending existing Combine-based APIs.
- Mark view models and shared state with `@MainActor`.
- Prefer `Codable` structs for data models; avoid classes unless reference semantics are needed.
- Use `NavigationSplitView`, `.sheet`, `.inspector`, and other native macOS SwiftUI APIs.
- Use Swift Package Manager for dependency management; do not use CocoaPods or Carthage.

### Python
- Target Python 3.10+. Use modern typing syntax (`dict[str, Any]`, `X | None` union syntax).
- Follow the existing handler pattern: functions take a param dict, return a result dict, raise on error.
- Use the `fdl` package for all FDL operations; do not duplicate its logic.
- Keep handlers stateless; the server manages no persistent state between requests.

### General
- Avoid unnecessary abstractions. Prefer simple, direct code.
- Use descriptive naming; avoid abbreviations except widely understood ones (FDL, JSON, SVG, etc.).
- Do not add comments that merely narrate what code does. Comments should explain non-obvious intent or constraints.
- When adding a new tool/feature, follow the existing pattern: create a View + ViewModel pair in a new feature directory under `Sources/FDLTool/`, and if backend work is needed, add a handler module in `python_backend/fdl_backend/handlers/` and register its methods in `server.py`.

## Build & Run
```bash
# Swift
cd FDLTool && swift build
swift run FDLTool

# Python backend (standalone)
cd python_backend && pip install -e ".[dev]"
python -m fdl_backend.server

# Tests
cd FDLTool && swift test
cd python_backend && pytest
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/davidbkhall) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
