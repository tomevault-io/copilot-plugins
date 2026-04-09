## omnitrack

> A standalone web app for tracking reading progress through omnibus books and their constituent child books.

# OmniTrack

A standalone web app for tracking reading progress through omnibus books and their constituent child books.

## Project Overview

OmniTrack models omnibus books (collected editions containing multiple books from a series) and calculates reading progress for each child book based on overall omnibus progress.

## Tech Stack

- Single HTML file (`index.html`) containing HTML, CSS, and JavaScript
- `sw.js` service worker for offline caching and PWA update support
- Local Storage for data persistence
- PWA compatible for iOS home screen installation

## Project Structure

```
OmniTrack/
├── index.html    # Complete app (HTML, CSS, JS)
├── sw.js         # Service worker (offline cache, PWA updates)
├── README.md     # User documentation
├── LICENSE       # MIT license
└── claude.md     # This file - project context for AI assistants
```

## Version

Current version: 1.0.7

## Data Model

### OmnibusBook (stored in localStorage)

| Property | Type | Description |
|----------|------|-------------|
| id | string | Unique identifier |
| name | string | Descriptive name of the omnibus |
| percentCompleted | number | User-entered progress (0-100) |
| pageCount | number | Total pages in the omnibus |
| books | array | Array of ChildBook objects |

### ChildBook

| Property | Type | Description |
|----------|------|-------------|
| name | string | Name of the child book |
| seriesNumber | number | Position in the series (used for sorting) |
| startPage | number | First page of this book in the omnibus |

### Calculated Properties (computed at render time)

**OmnibusBook.pagesRead:** `percentCompleted * pageCount / 100`

**ChildBook.endPage:**
- If last child in omnibus: `OmnibusBook.pageCount`
- Otherwise: `NextChildBook.startPage - 1`

**ChildBook.totalPages:** `endPage - startPage + 1`

**ChildBook.percentOfBook:** `totalPages / OmnibusBook.pageCount`

**ChildBook.pagesRead:**
- If `OmnibusBook.pagesRead >= ChildBook.endPage`: `ChildBook.totalPages` (completed)
- Else if `OmnibusBook.pagesRead < ChildBook.startPage`: `0` (not started)
- Else: `OmnibusBook.pagesRead - ChildBook.startPage + 1` (in progress)

**ChildBook.percentRead:**
- If `ChildBook.pagesRead <= 0`: `0`
- Else: `ChildBook.pagesRead / ChildBook.totalPages`

## Key Functions

- `loadData()` / `saveData()` - localStorage persistence
- `calculatePagesRead(omnibus)` - calculates pages read from percent
- `calculateChildBookProperties(omnibus)` - computes all child book derived values
- `render()` - renders all omnibus cards to DOM
- `adjustProgress(id, increment)` - +/- button handler
- `showCreateModal()` / `editOmnibus(id)` - modal management
- `addChildBookForm()` - dynamically adds child book input fields (Enter key adds another)
- `exportData()` / `importData()` / `handleImport()` - JSON export/import of full OmniTrack backup
- `importOmnibusFile()` / `handleOmnibusImport()` - import a single omnibus from ebook-derived JSON
- `resetData()` - clears all data (with confirmation)

## Omnibus Import Format

JSON files produced by the companion ebook reader app can be imported via "Import Book" in settings:

```json
{
  "omnibus_title": "string",
  "total_pages": number,
  "page_count_method": "string (informational, ignored)",
  "child_books": [
    { "title": "string", "start_page": number }
  ]
}
```

`seriesNumber` is assigned from array order (1-based). `percentCompleted` defaults to 0.

## UI Components

- **Main view**: List of omnibus cards with progress bars, gear icon for settings
- **Omnibus modal**: Create/edit omnibus with inline child book forms
- **Child modal**: Edit individual child book
- **Confirm modal**: Delete/reset confirmation dialog
- **Settings modal**: Export, import, and reset data options

## localStorage

Data stored under key `omnitrack_data` as JSON array of OmnibusBook objects.

## Export Format

Exported JSON filename: `omnitrack-export-YYYY-MM-DDTHH-MM-SS.json`

The export contains the raw `omnibusBooks` array, which can be re-imported to restore data.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/wyldphyre)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/wyldphyre)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
