## markdown-ohp

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a static web-based markdown viewer application that runs entirely in the browser. It requires no build process or server-side functionality.

## Commands

This is a static website with no build tools:
- **Run locally**: `python -m http.server 8000` (or any HTTP server)
- **Deploy**: Simply host the files on any static web server
- **No build/lint/test commands** - Pure HTML/CSS/JS application

## Architecture

### Current Architecture (Web Components)
The application uses native Web Components with Shadow DOM for encapsulation. All components are defined in a single file to avoid ES6 module MIME type issues.

#### Web Components (web-components.js)
- **MarkdownViewerApp**: Main application component that manages state and routing
- **DropZoneComponent**: File drop area with drag/drop and file picker support
- **ViewerContainer**: Container for the markdown viewer with header controls
- **MarkdownContent**: Renders markdown content with syntax highlighting and diagrams
- **TocNavigation**: Table of contents sidebar with active heading tracking
- **ImageLightbox**: Dialog-based image preview for images and Mermaid diagrams

#### Key Files
- **web-components.js**: All Web Components in a single file
- **index.html**: Minimal HTML that loads the components
- **styles.css**: Removed - styles are now encapsulated in Shadow DOM
- **print.css**: Removed - print styles handled by media queries

#### VSCode Extension (vscode-markdown-viewer/)
- **extension.js**: VSCode extension that creates WebView for markdown preview
- **package.json**: Extension manifest and configuration

#### Legacy Files (old/)
Previous modular architecture preserved for reference

### External Dependencies (CDN)
- **marked.js**: Markdown parsing and rendering
- **Mermaid**: Diagram rendering (flowcharts, sequence diagrams, etc.)
- **highlight.js**: Syntax highlighting for code blocks

### Key Features Implementation
1. **Hot Reload**: Uses File System Access API (Chrome/Edge only) to watch file changes
2. **Drag & Drop**: Standard HTML5 drag/drop API with fallback file picker
3. **Table of Contents**: Dynamically generated from markdown headings with scroll tracking
4. **Collapsible Sections**: Custom implementation that wraps content between headings
5. **Lightbox**: Native HTML dialog elements for images and diagrams
6. **PDF Export**: Browser print functionality with custom print styles
7. **Session Persistence**: Uses localStorage to save and restore file content across page reloads (with 24-hour expiration)
8. **Front Matter Support**: Parses YAML front matter from markdown files
   - Supports title, author, date, tags, and custom metadata
   - Front Matter title takes precedence for page title
   - UI toggle button to show/hide metadata (hidden by default)

### Browser Compatibility
- Full features: Chrome/Edge (File System Access API support)
- Basic features: Firefox, Safari (no hot reload)

### Deployment
The app is deployed to GitHub Pages at https://monoharada.github.io/markdown-ohp/

## Testing Guide

### Local Testing
1. Open `index.html` in a web browser
2. Drag and drop `front-matter-sample.md` to test Front Matter support
3. Click the metadata toggle button (leftmost button in header) to show/hide Front Matter
4. Note: The page title should display the Front Matter title if present

### Known Issues
- Session restore shows "Untitled.md" instead of original filename (localStorage limitation)
- Front Matter parsing is basic - complex YAML structures not supported

---
> Source: [monoharada/markdown-ohp](https://github.com/monoharada/markdown-ohp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-13 -->
