## slide-web-app

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Development Commands

- `npm run dev` - Start development server (default port 5173)
- `npm run build` - Build for production (TypeScript compilation + Vite build)
- `npm run lint` - Run ESLint
- `npm run preview` - Preview production build

## Project Overview

**SlideForge Pro** is a comprehensive React-based slide editor application with rich text editing, drawing capabilities, and professional export functionality. Built using modern web technologies, the application provides a complete presentation creation experience with support for various element types, transformation controls, and professional output formats (PDF/PPTX).

## Core Architecture

### State Management (Jotai + Immer)
- **Central State**: All application state managed through Jotai atoms in `src/state/atoms.ts`
- **Document Model**: Hierarchical structure (Document → Slides → Elements) defined in `src/domain/models.ts`
- **Immutable Updates**: Immer with patches enabled for undo/redo functionality
- **Key Atoms**:
  - `documentAtom` - Main document state
  - `currentSlideIdAtom` - Active slide tracking
  - `selectionAtom` - Element selection state
  - Text editing atoms for rich text functionality

### Canvas System (React-Konva)
- **Canvas Stage**: Primary rendering through `CanvasStage.tsx` using Konva.js
- **Element Rendering**: `LayerElements.tsx` handles rendering of all slide elements (text, shapes, arrows, lines)
- **Transformation**: `TransformerOverlay.tsx` provides resize/rotate handles with enhanced multi-selection support
- **Selection System**: Supports both individual and multiple element selection with drag selection
- **Guidelines**: `GuideLines.tsx` provides snap-to-align functionality during element manipulation
- **Text Elements**: Support word wrapping, inline editing, proper scaling behavior, and rich text editing

### Data Flow Architecture
- **Domain Actions**: `src/domain/actions.ts` contains all state mutation functions
- **Immer Integration**: All updates use `produceWithPatches` for automatic undo/redo history
- **Patch-Based History**: Undo/redo system tracks forward/backward patches
- **React Integration**: Components subscribe to atoms for reactive updates

### Text Editing System
- **Double-click Activation**: Text elements enter edit mode on double-click
- **Rich Text Toolbar**: `TextToolbar.tsx` provides font family, size, color, weight, alignment controls
- **Inline Text Editing**: `InlineTextEditor.tsx` provides direct text editing within canvas
- **Smart Positioning**: Toolbar automatically positions above/below text to avoid overlap
- **Real-time Updates**: Changes reflect immediately in canvas through atom synchronization

### Export System
- **PDF Export**: `src/utils/pdfExport.ts` handles PDF generation via jsPDF with accurate canvas-to-PDF conversion
- **PPTX Export**: `src/utils/pptxExport.ts` handles PowerPoint export via PptxGenJS with element-by-element conversion
- **Aspect Ratio Support**: Both exporters support 4:3 and 16:9 aspect ratios
- **High Fidelity**: Coordinate conversion, color mapping, and font handling for accurate reproduction

## Key Technical Implementations

### Element Transformation Handling
- **Text Elements**: Special transformation logic maintains font size during resize operations (unlike shapes which scale proportionally)
- **Multi-Selection**: Enhanced `TransformerOverlay.tsx` supports simultaneous transformation of multiple elements
- **Shape Elements**: Proper handling of different shape types (rectangles, circles, triangles, arrows, lines) with appropriate constraints
- **Arrow Handling**: Specialized arrow rendering that maintains proportions and arrow head sizing

### Canvas Event System
- **Selection Handling**: Click/tap for individual selection, Ctrl+click for multi-selection
- **Drag Selection**: Rectangle selection tool for selecting multiple elements
- **Text Editing**: Double-click activation for text editing with inline editor
- **Element Manipulation**: Drag handling for element positioning with snap guidelines
- **Transformation**: Transform end events for size/position updates with multi-element support
- **Clipboard Operations**: Copy/paste functionality with Ctrl+C/Ctrl+V keyboard shortcuts

### Immer Patches Requirement
The application requires `enablePatches()` to be called in `main.tsx` for the undo/redo system to function. This enables Immer to generate forward and inverse patches for all state mutations.

## Component Hierarchy

```
App.tsx (main application with 2-tier toolbar + title header)
├── CanvasStage.tsx (Konva Stage wrapper with export capabilities)
│   ├── LayerElements.tsx (renders all slide elements)
│   ├── TransformerOverlay.tsx (selection/resize handles with multi-selection)
│   ├── GuideLines.tsx (snap-to-align guide lines)
│   ├── SelectionBox.tsx (drag selection rectangle)
│   ├── TextToolbar.tsx (rich text editing toolbar, positioned absolutely)
│   ├── ShapeToolbar.tsx (shape editing controls)
│   └── InlineTextEditor.tsx (direct text editing within canvas)
├── HelpModal.tsx (help dialog with keyboard shortcuts)
└── useHotkeys.ts (keyboard shortcuts for undo/redo/copy/paste)
```

## UI Structure

### Header (App Title)
- **SlideForge Pro** branding with professional dark theme
- Subtitle indicating "プロフェッショナル スライド エディター"

### 2-Tier Toolbar Layout
**First Tier:**
- Object creation buttons (Rectangle, Circle, Triangle, Arrow, Line, Text)
- Action buttons (Undo, Redo, Clear Selection)

**Second Tier:**
- Aspect ratio controls (4:3, 16:9)
- Export buttons (PDF, PPTX)
- Help button

## Element Types & Properties

### Base Element Properties
All elements extend a base type with:
- **Position**: x, y coordinates
- **Size**: width (w), height (h)
- **Layer**: z-index for stacking order
- **Transformation**: rotation support
- **Visual**: opacity and lock state

### Text Elements
- **Font Properties**: family, size, color, weight
- **Alignment**: left, center, right
- **Text Content**: with word wrapping support
- **Inline Editing**: Direct text editing within canvas

### Shape Elements
- **Visual Properties**: fill colors, stroke colors, stroke width
- **Shape-Specific**: corner radius for rectangles
- **Supported Shapes**: rectangles, circles, triangles, arrows, lines
- **Arrow Rendering**: Special handling with proper arrow heads

## Dependencies

### Core Libraries
- **React 19.1.1** - UI framework
- **TypeScript** - Type safety
- **Vite** - Build tool and dev server

### State Management
- **Jotai 2.13.1** - Atomic state management
- **Immer 10.1.1** - Immutable updates with patches

### Canvas & Graphics
- **Konva 9.3.22** - Canvas rendering engine
- **React-Konva 19.0.7** - React bindings for Konva

### Export Functionality
- **jsPDF 3.0.2** - PDF generation
- **PptxGenJS 4.0.1** - PowerPoint generation

### Utilities
- **nanoid 5.1.5** - Unique ID generation

## Export Features

### PDF Export
- Canvas-to-image conversion with high quality
- Preserves exact visual appearance
- Support for different aspect ratios
- Maintains resolution and color fidelity

### PPTX Export
- Element-by-element conversion to PowerPoint objects
- Native PowerPoint shapes (rectangles, circles, triangles)
- Line-based arrows with PowerPoint's built-in arrow heads
- Font mapping to Web Safe fonts for compatibility
- Color conversion with proper hex-to-PowerPoint format
- Coordinate system conversion (px to inches)
- Editable output - users can modify in PowerPoint

## Known Considerations

### PPTX Export Limitations
- Visual differences between browser and PowerPoint are expected due to different rendering engines
- Font rendering may vary slightly between platforms
- Complex shapes are simplified to basic PowerPoint equivalents
- Arrow rendering uses PowerPoint's native line arrows for consistency

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/shimpei1494)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/shimpei1494)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
