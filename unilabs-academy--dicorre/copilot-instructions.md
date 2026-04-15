## dicorre

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a browser-based DICOM anonymization and sending tool. Key features:
- Client-side DICOM file processing and anonymization
- DICOMweb-based sending to radiology destinations
- Plugin architecture for format converters
- Project-based configurations for sharing with contributors

## Tech Stack Recommendations

Based on PRD requirements:
- **Language**: Typescript
- **Frontend Framework**: Vue.js for browser-based UI
- **DICOM Processing**: dcmjs library
- **Anonymization**: @umessen/dicom-deidentifier (TypeScript-friendly, standard profiles)
- **DICOM Sending**: dicomweb-client
- **File Handling**: JSZip for unzipping files in browser
- **Build Tool**: Vite

### UI
- We use vue shadcn component system
- Components located in /aca_nuxt/layers/base/components/ui
- Components are auto-imported with Ui prefic (UiButton) but prefer explicit imports except for small common ones like button.
- Component install via shadcn cli currently broken due to folder layout so ask if necessary to install new component

### Styling
- Always use tailwind for styling
- We are currently using tailwind 3 so variables are tracked both in tailwind.css and tailwind.config.js
- Prefer variables in aca_nuxt/layers/base/assets/css/tailwind.css especially for color
- primary-light-gray, secondary-gray and primary-dark-gray most commonly used in UI (sometimes with /50 (50 opacity if subtle change is needed))
- It's ONLY ok to write scoped css in a vue component if there isn't a nice way to solve it with tailwind (like grids with named slots or tables when a @apply utility class can save a lot of duplicates)

## Development Commands

Once the project is initialized, common commands will be:

```bash
# Install dependencies
pnpm install

# Run development server
pnpm dev

# Build for production
pnpm build

# Run tests
pnpm test

# Lint code
pnpm lint

# Type checking
pnpm type-check

# Run Docker compose for dcm4chee test server
docker-compose up -d
```

## Architecture Guidelines

TODO

## Testing

When testing the web app with Playwright e2e test or using the Playwright MCP:

### NEVER USE TIMEOUTS
- Never use timouts to wait for processes to complete
- Prefer playright tools like { waitUntil: 'networkIdle' } or waiting for expected UI state

### File Upload Testing
- **Issue**: File uploads require clicking the file label to trigger the browser's file chooser
- **Solution**:
  1. Click the "Browse Files" label element to open file chooser
  2. Use `browser_file_upload` tool when modal state shows "[File chooser]"
  3. Example workflow:
     ```typescript
     // Click Browse Files label to trigger file input
     await page.getByText('Browse Files').click()
     // Wait for file chooser modal state
     // Then upload file
     await fileChooser.setFiles(['/path/to/test/file.zip'])
     ```

### DICOM Anonymization Testing
- **Important**: The `@umessen/dicom-deidentifier` library expects raw `Uint8Array` data, not parsed dcmjs datasets
- **Correct usage**: Pass `new Uint8Array(arrayBuffer)` directly to `deidentifier.deidentify()`
- **Test files**: Use `/test-data/CASES/Caso1.zip` which contains valid DICOM files with proper headers

### Console Monitoring
- Use `browser_console_messages()` to monitor application logs and errors
- Helpful for debugging DICOM processing and anonymization issues

### Scripts
- Use scripts when you need to examing data like dicom files
- Scripts can be written in either python or typescript
- You find examples of .py and .ts scripts in the /scripts folder
- Delete scripts if only used temporarily but keet them if they can be useful for reference in the future
- For python scripts use uv to run with requirements in header as in examples

# Documentation
-----------------
[!] ALWAYS REFER TO DOCUMENTATION:
Vue: https://vuejs.org/llms.txt
Effect: https://effect.website/llms.txt
Shadcn-vue: https://www.shadcn-vue.com/llms.txt
- always use pnpm instead of npm
- never try to run devserver yourself unless specifically asked
- dev server already running on port 5173

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Unilabs-Academy)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Unilabs-Academy)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
