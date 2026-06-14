## sqliteer

> SQLiteER is a macOS SwiftUI app for opening SQLite database files, reading their table schemas, and displaying an entity relationship diagram. The first usable version focuses on schema-level diagrams: tables, columns, primary keys, foreign keys, row counts, and inferred joins.

# SQLiteER Project Notes

## Project Overview
SQLiteER is a macOS SwiftUI app for opening SQLite database files, reading their table schemas, and displaying an entity relationship diagram. The first usable version focuses on schema-level diagrams: tables, columns, primary keys, foreign keys, row counts, and inferred joins.

## Architecture
- `ContentView` owns UI state and coordinates file import/export.
- `SQLiteSchemaLoader` reads SQLite metadata through the system SQLite C API.
- `DatabaseSchema`, `TableSchema`, `ColumnSchema`, and `TableRelationship` model the database independently from the UI.
- `DiagramLayout` computes stable table positions and connector anchors.
- `ERDiagramView` renders the diagram and is reused for both on-screen display and image/PDF export.

## Conventions
- Prefer modern SwiftUI and Observation (`@Observable`) for app state.
- Keep SQLite access isolated from views.
- Keep export rendering in one path so PNG and PDF output match the visible diagram.
- Avoid force unwraps; surface recoverable failures as user-visible messages.

## Build and Run
- Active scheme: `SQLiteER`
- Platform: macOS
- Build with Xcode or the `BuildProject` MCP command.
- The app is sandboxed and requires user-selected file access for opening SQLite files and saving exports.

## Gotchas
- SQLite foreign keys can be disabled at runtime, but `PRAGMA foreign_key_list` still exposes declared relationships.
- Many databases do not declare foreign keys. The app also infers likely joins from columns named `*_id` or matching another table's primary key.
- Export uses `ImageRenderer`; very large schemas may produce large PNG/PDF files.

---
> Source: [StewartLynch/SQLiteER](https://github.com/StewartLynch/SQLiteER) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-14 -->
