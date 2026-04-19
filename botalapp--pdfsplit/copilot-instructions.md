## pdfsplit

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

PDFSplit is a modern, client-side PDF splitting tool built with vanilla JavaScript and modern web standards. It provides a complete solution for splitting PDF files with multiple modes, drag-and-drop upload, real-time preview, and batch download functionality.

## Development Commands

### Setup and Installation
```bash
npm install                 # Install all dependencies
```

### Development
```bash
npm run dev                 # Start development server (localhost:3000)
npm run preview            # Preview built application (localhost:3001)
```

### Build and Production
```bash
npm run build              # Build for production (builds both CSS and JS)
npm run build:css          # Build CSS only (with PostCSS processing)
npm run build:js           # Build JavaScript only (with ESBuild)
```

### Code Quality
```bash
npm run lint               # Run ESLint on JavaScript files
npm run format             # Format code with Prettier
```

## Architecture

### Core Structure
- **Client-side Processing**: All PDF operations happen in the browser for privacy
- **Modular Architecture**: Separated into focused modules for maintainability
- **No Backend Required**: Static deployment with CDN-hosted dependencies

### Key Modules

#### 1. Main Application (`src/js/main.js`)
- Central application controller
- Event handling and coordination between modules
- Application state management

#### 2. File Handler (`src/js/modules/fileHandler.js`)
- File validation (type, size, format)
- File reading and ArrayBuffer conversion
- Filename generation and blob URL management

#### 3. PDF Processor (`src/js/modules/pdfProcessor.js`)
- PDF loading using pdf-lib and PDF.js
- Page rendering for thumbnails
- PDF splitting logic for all modes:
  - Split by pages per file
  - Split by page ranges
  - Extract specific pages

#### 4. UI Controller (`src/js/modules/uiController.js`)
- Progress indicators and status updates
- Section visibility management
- Error handling and user feedback
- Form validation

#### 5. Download Manager (`src/js/modules/downloadManager.js`)
- Blob URL creation and cleanup
- Batch download functionality
- File size calculations and formatting

### Technology Stack
- **PDF Processing**: pdf-lib (manipulation), PDF.js (rendering)
- **Build Tools**: ESBuild (JS bundling), PostCSS (CSS processing)
- **Development**: live-server (dev server), ESLint, Prettier
- **Deployment**: Netlify (with optimized headers and redirects)

### File Processing Flow
1. User uploads PDF file (drag-and-drop or click)
2. File validation (type, size, format)
3. PDF loading with both pdf-lib and PDF.js
4. Thumbnail generation for preview
5. User selects split mode and options
6. PDF processing based on selected mode
7. Download links generation
8. File cleanup and memory management

## Deployment

### Netlify Deployment
The application is optimized for Netlify deployment with:
- `netlify.toml`: Build settings, headers, redirects
- `_redirects`: SPA routing and cache headers
- Security headers: CSP, X-Frame-Options, etc.
- Performance optimizations: asset caching, compression

### Build Process
1. CSS processing with PostCSS (autoprefixer, cssnano)
2. JavaScript bundling with ESBuild (minification, ES2020 target)
3. Static file optimization
4. Security header configuration

## Key Features Implementation

### Split Modes
- **By Pages**: Split into files with N pages each
- **By Ranges**: Split by user-defined page ranges (e.g., "1-3, 5-10, 12")
- **Extract Pages**: Extract specific pages (e.g., "1, 3, 5, 8")

### UI/UX Features
- Responsive design (mobile-first approach)
- Drag-and-drop file upload with visual feedback
- Real-time progress indicators
- Thumbnail grid with page selection
- Animated transitions and loading states
- Error handling with user-friendly messages

### Performance Optimizations
- Lazy loading of PDF pages
- Memory management (blob URL cleanup)
- Efficient canvas rendering
- Progressive enhancement approach

## Browser Compatibility
- Modern browsers with ES6+ module support
- Chrome 90+, Firefox 88+, Safari 14+, Edge 90+
- WebAssembly support required for PDF.js

## Security Considerations
- All processing happens client-side (no server uploads)
- Strict Content Security Policy
- HTTPS enforcement
- XSS protection headers
- File type validation

## Development Notes
- Uses ES6+ modules with dynamic imports
- No build step required for development
- CDN dependencies for pdf-lib and PDF.js
- Modern CSS with custom properties (CSS variables)
- Progressive enhancement for better accessibility

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/botalapp) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
