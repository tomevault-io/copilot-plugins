## simplehtmleditor

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

HTML Position Editor - A visual drag-and-drop editor for adjusting element positions in fixed-dimension HTML presentations (375×667px). The editor uses CSS transforms for positioning without modifying the content structure.

## Project Structure

```
simplehtmleditor/
├── index.html              # Single-file application (HTML, CSS, JavaScript)
├── html/                   # Sample HTML files
│   └── sample01.html      
├── docs/
│   └── html-editor-requirements.md
├── README.md
├── LICENSE
└── CLAUDE.md              # This file
```

## Architecture

### Single-File Application
The entire application is contained within `index.html` with inline CSS and JavaScript. The code is organized into logical sections:

1. **HTML Structure**: Editor UI with toolbar, preview frame, and layer panel
2. **CSS Styles**: Organized with CSS variables for theming
3. **JavaScript Application**: Object-based architecture with the following modules:
   - `EditorApp` - Main application object
   - `fileManager` - File operations (open/save)
   - `pageManager` - Multi-page navigation
   - `elementManager` - Element selection and manipulation
   - `historyManager` - Undo/redo functionality
   - `exportManager` - PDF/SVG export
   - `viewManager` - View controls (zoom/grid/snap)
   - `eventHandler` - Event handling

### Key Implementation Details

- **Module Pattern**: Each feature is implemented as a method on the EditorApp object
- **State Management**: Centralized in `EditorApp.state` object
- **DOM Caching**: Elements cached in `EditorApp.elements` on initialization
- **Event Delegation**: iframe content manipulation through contentDocument
- **CSS Transforms**: Used for element positioning to preserve original layout

## Running the Application

```bash
# Development server (recommended for proper CORS handling)
python -m http.server 8000
# Navigate to http://localhost:8000/

# Or use any other local server:
# - VS Code Live Server
# - Node.js http-server
# - PHP built-in server
```

## Common Development Tasks

### Testing Export Functions
```javascript
// Test PDF export (print)
EditorApp.exportManager.exportToPDF()

// Test PDF export (image)
EditorApp.exportManager.exportToPDFImage()

// Test SVG export
EditorApp.exportManager.exportToSVG()
```

### Debugging Element Selection
```javascript
// Check current selection
console.log(EditorApp.state.selectedElement)

// List all movable elements
console.log(EditorApp.state.movableElements)
```

## External Dependencies

Loaded from CDN:
- **jsPDF** (2.5.1) - PDF generation
- **html2canvas** (1.4.1) - HTML to canvas conversion
- **JSZip** (3.10.1) - SVG export packaging

## Browser Requirements
- Modern browsers with ES6 support
- File System Access API (Chrome/Edge 86+) for enhanced save functionality
- Fallback file operations for other browsers

## Known Issues and Solutions

### Export Issues
- **PDF Export**: Uses browser print dialog or html2canvas for image export
- **SVG Export**: Converts HTML to SVG with foreignObject
- **Clean Export**: Temporary removal of editor UI elements during export

### Cross-Frame Communication
- Element selection through iframe requires contentDocument access
- Event listeners must be re-attached when iframe content changes

## Key Functions and Entry Points

- **Initialization**: `EditorApp.init()` in DOMContentLoaded event
- **File Operations**: 
  - `EditorApp.fileManager.openFile()`
  - `EditorApp.fileManager.saveFile()`
- **Export Functions**:
  - `EditorApp.exportManager.exportToPDF()` 
  - `EditorApp.exportManager.exportToPDFImage()`
  - `EditorApp.exportManager.exportToSVG()`
- **Element Management**:
  - `EditorApp.elementManager.selectElement(element)`
  - `EditorApp.elementManager.startDrag(event)`
  - `EditorApp.elementManager.startResize(event, handle)`

## Debugging Commands

```javascript
// View current state
console.log(EditorApp.state)

// Check frame document
console.log(EditorApp.getFrameDocument())

// Force history save
EditorApp.historyManager.saveState()

// Clear selection
EditorApp.elementManager.clearSelection()
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/moshimoshipandasan) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
