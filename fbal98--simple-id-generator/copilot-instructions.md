## simple-id-generator

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Simple ID Generator is a web application that allows users to:
1. Upload custom ID template images
2. Add customizable text and photo fields to the template
3. Generate multiple ID cards with randomized data
4. Download individual IDs or all generated IDs as a ZIP archive

## Commands

### Running the Application
```bash
# Install dependencies
bun install

# Start the development server
bun start
```

The application will be available at http://localhost:3000

### Testing

The project includes comprehensive E2E tests using Playwright:

```bash
# Install test dependencies
bun run test:install

# Quick smoke test (7 tests, ~3s) - Fastest for development
bun run test:smoke

# Core functionality tests (28 tests, ~10s) - Good for CI/pre-commit
bun run test:core

# Fast full test suite (405 tests, ~30s) - Chromium only
bun run test:fast

# Full test suite across all browsers (~2-3 minutes)
bun test

# Individual browser testing
bun run test:chromium
bun run test:firefox
bun run test:webkit

# Debug mode for test development
bun run test:debug
bun run test:ui
```

**Test Coverage:**
- Template upload and validation
- Field management (drag, resize, styling)
- ID generation with random data
- Download functionality (PNG/ZIP)
- Server proxy and API endpoints
- Error scenarios and edge cases
- Cross-browser compatibility

## Architecture

The application consists of a Bun-based server for static file serving and API proxy functionality, with a vanilla JavaScript frontend.

### Core Components

1. **Server (`server.js`)**
   - Serves static files (HTML, JS, CSS)
   - Provides a proxy endpoint `/api/face` to fetch AI-generated face images from thispersondoesnotexist.com
   - Handles CORS and basic security measures

2. **Frontend Modules**
   - **app.js**: Main application logic, manages app state and connects UI components
   - **fieldManager.js**: Handles draggable/resizable field overlay management
   - **dataGenerator.js**: Contains functions for generating random data for IDs

3. **UI Structure**
   - Left panel: Controls for template upload and field addition
   - Center: Canvas for ID template with draggable field overlays
   - Right panel: Controls for ID generation and download

### Data Flow

1. User uploads a template image and adds fields to it
2. When generating IDs, the app:
   - Creates random data for each field
   - Fetches AI faces for photo fields via the server proxy
   - Renders each ID to an offscreen canvas
   - Converts to PNG data URLs
   - Provides download options for individual or bulk ZIP downloads

### Key Features

1. **Field Management**
   - Add, position, and resize various field types (name, dates, civil number, photo)
   - Custom styling for each field type
   - Real-time positioning and updates

2. **ID Generation**
   - Generates random, culturally appropriate names
   - Creates plausible dates and civil numbers
   - Fetches AI-generated face photos
   - Supports batch generation with progress indication

3. **Export Options**
   - Download individual preview
   - Download all as ZIP (using JSZip from CDN)
   - Edit layout after generation

## File Structure

```
.
├── index.html                    # Main HTML file
├── server.js                     # Bun server for static files and proxying AI face fetch
├── package.json                  # Project metadata and scripts
├── bun.lockb                     # Bun lockfile
├── playwright.config.js          # Playwright test configuration
├── E2E_TESTING_CHECKLIST.md     # Comprehensive testing documentation
├── styles/
│   └── main.css                  # Stylesheet
├── js/
│   ├── app.js                    # Front-end application logic
│   ├── fieldManager.js           # Draggable/resizable field manager
│   └── dataGenerator.js          # Random data generator functions
└── tests/                        # E2E test suite (405 tests)
    ├── fixtures/
    │   └── realistic-id-template.svg  # SVG test template based on Omani ID structure
    ├── template-upload.spec.js    # Template upload functionality tests
    ├── field-management.spec.js   # Field drag/resize/styling tests
    ├── id-generation.spec.js      # ID generation and data validation tests
    ├── download.spec.js           # Download PNG/ZIP functionality tests
    ├── server-proxy.spec.js       # Server API and proxy endpoint tests
    ├── error-scenarios.spec.js    # Error handling and edge case tests
    └── full-workflow.spec.js      # End-to-end integration workflow tests
```

## Testing Strategy

The project uses a multi-layered testing approach:

1. **Smoke Tests** (`test:smoke`) - Core functionality validation for rapid development feedback
2. **Core Tests** (`test:core`) - Essential features for CI/CD pipelines
3. **Full Suite** (`test:fast`) - Comprehensive testing optimized for speed
4. **Cross-Browser** (`test`) - Full compatibility validation across Chromium, Firefox, and WebKit

**Performance Optimizations:**
- Disabled video/screenshot recording for speed
- Reduced timeouts (5s actions, 10s navigation)
- Browser launch optimizations with security flags
- Parallel test execution with configurable workers
- Dot reporter for minimal output during fast runs

---
> Source: [fbal98/simple-id-generator](https://github.com/fbal98/simple-id-generator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-15 -->
