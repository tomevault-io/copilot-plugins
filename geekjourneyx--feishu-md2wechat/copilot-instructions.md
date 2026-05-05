## feishu-md2wechat

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 📚 Essential Documentation

**🎯 BEFORE starting ANY development task, please review these documents:**

- **[UI Design Guide](./docs/UI-DESIGN-GUIDE.md)** - Complete visual design system, color schemes, component specifications, and responsive design guidelines
- **[Development Guide](./docs/DEVELOPMENT-GUIDE.md)** - Code style standards, file organization, Chrome extension patterns, and best practices
- **[Icon Design Prompts](./docs/ICON-DESIGN-PROMPTS.md)** - Professional icon design prompts for brand identity, optimized for Apple Design aesthetics and multiple sizes

These guides contain critical lessons learned and established patterns that ensure consistency and prevent common pitfalls.

## Project Overview

Feishu Converter is a Chrome browser extension (also available for Edge and Firefox) that converts Lark/Feishu documents to Markdown format. The project supports downloading, copying, and previewing documents with WeChat formatting capabilities.

## Development Commands

### Essential Commands
```bash
# Install dependencies (run from root)
pnpm i

# Build the entire project
pnpm run build

# Build only the Chrome extension
cd apps/chrome-extension && pnpm run build

# Development build (non-minified)
cd apps/chrome-extension && pnpm run build:dev

# Type checking
pnpm run type-check

# Linting
pnpm run lint

# Format code
pnpm run format

# Run tests
pnpm run test

# Clean build artifacts
pnpm run clean
```

### Chrome Extension Specific
```bash
# Build Firefox version
cd apps/chrome-extension && pnpm run build:firefox

# Copy assets to dist folder
cd apps/chrome-extension && npm run copy
```

## Architecture Overview

### Monorepo Structure
This is a pnpm workspace monorepo with Turbo for build orchestration:
- **apps/chrome-extension**: Main Chrome extension application
- **packages/common**: Shared utilities (DOM manipulation, time, image processing)
- **packages/lark**: Core Lark document processing and conversion logic
- **packages/rollup-config**: Shared Rollup build configuration
- **packages/typescript-config**: Shared TypeScript configurations

### Chrome Extension Architecture

The extension follows Chrome Manifest V3 architecture with distinct runtime contexts:

#### Core Components
1. **Background Script** (`src/background.ts`): Service worker handling context menus, tab management, and message routing
2. **Content Script** (`src/content.ts`): Injected into Lark pages for message passing between page and extension
3. **Sidebar Panel**: Main UI using Chrome's sidePanel API with multiple views:
   - Main sidebar (`src/sidebar/sidebar.ts`)
   - Settings page (`src/sidebar/settings.ts`)
   - WeChat preview (`src/sidebar/wechat-preview.ts`)
4. **Injected Scripts** (`src/scripts/`): Run in MAIN world context to access Lark's document APIs

#### Message Passing System
- Uses Chrome runtime messaging between background, content script, and sidebar
- window.postMessage for communication between injected scripts and content script
- Message types defined in `src/common/message.ts` with typed interfaces

#### Supported Document Sources
- *.feishu.cn, *.feishu.net, *.larksuite.com
- *.feishu-pre.net, *.larkoffice.com, *.larkenterprise.com

### Document Processing Pipeline

1. **Extraction**: Injected scripts access Lark's internal document structure
2. **Conversion**: `@dolphin/lark` package converts to MDAST (Markdown AST)
3. **Serialization**: MDAST converted to Markdown using mdast-util-to-markdown
4. **Output**: Download as file, copy to clipboard, or preview with WeChat formatting

### WeChat Integration
- MD2WeChat API integration for WeChat article formatting
- Custom CSS styling matching WeChat's design system
- Full-screen preview with floating toolbar interface
- Copy functionality optimized for WeChat backend editor compatibility

## Key Technical Details

### Build System
- **Rollup** for bundling with format-specific outputs:
  - ESM format for background/content scripts
  - IIFE format for UI scripts and injected scripts
- **Babel** for browser compatibility
- **TypeScript** with multiple tsconfig files for different runtime contexts

### Styling & Design
- Primary color scheme: `#d97759` (warm orange)
- CSS custom properties for consistent theming
- Full-screen responsive design for WeChat preview
- Modern UI with backdrop blur effects and smooth animations

### Internationalization
- i18next for translations
- Supports English (`en`) and Chinese (`zh_CN`)
- Chrome extension `_locales` structure

### State Management
- Chrome storage API for user preferences (API keys, themes, font sizes)
- No external state management library; uses local component state

## Development Notes

### File Naming Conventions
- TypeScript files use `.ts` extension throughout
- Build outputs generate corresponding `.js` files in `bundles/` directory
- HTML files are copied directly to `dist/` during build

### Extension Permissions
- contextMenus, scripting, sidePanel, tabs, storage
- Host permissions for all Lark/Feishu domains
- Content scripts inject only on document_end

### Testing
- Vitest for unit tests
- Test files follow `*.test.ts` pattern
- Workspace configuration supports testing across all packages

### Code Quality
- ESLint for linting with TypeScript rules
- Prettier for code formatting
- Husky + lint-staged for pre-commit hooks
- Type checking enforced across all packages

## Development Best Practices & Lessons Learned

### Critical Checks Before Development

#### 1. File Type Consistency
**ALWAYS** ensure proper file extensions:
- Use `.ts` for ALL source files in the `src/` directory
- Never mix `.js` and `.ts` files in the same project context
- The entire `src/` directory uses TypeScript compilation; `.js` files won't be processed

#### 2. Build Configuration Verification
- Verify rollup.config.js includes new files in appropriate entry points
- Check that new scripts are added to the glob pattern: `src/scripts/*.ts`
- Ensure correct module format (ESM vs IIFE) based on runtime context:
  - Background/content scripts: ESM format
  - UI scripts (sidebar, popup): IIFE format for HTML script tag compatibility

#### 3. Message Passing Architecture
- Update `src/common/message.ts` when adding new message types
- Maintain proper TypeScript interfaces for all message payloads
- Test communication chain: sidebar → background → content script → injected script

### Common Development Pitfalls

#### 1. Chrome Extension Context Issues
- **Injected scripts** (MAIN world) cannot access `chrome.runtime` APIs
- Use `window.postMessage` to communicate from injected scripts to content script
- Content script acts as bridge between page context and extension context

#### 2. Module Format Problems
- HTML script tags require IIFE format, not ESM modules
- Rollup configuration separates these concerns automatically
- Test in browser console if scripts load properly

#### 3. Color Theme Consistency
- Primary color: `#d97759` (warm orange)
- Replace ALL success states from green (`#22c55e`, `#34c759`) to primary color
- Use CSS custom properties (`var(--primary-color)`) for consistency
- Check notification backgrounds, button states, and status indicators

### Adding New Features

#### 1. New Sidebar Pages
1. Create HTML file in `src/sidebar/`
2. Create corresponding `.ts` file
3. Add to rollup.config.js IIFE outputs
4. Update copy script in package.json if needed
5. Add navigation logic in existing sidebar components

#### 2. New Document Processing Features
1. Extend types in `packages/lark/src/docx.ts`
2. Add processing logic in appropriate conversion functions
3. Update MDAST interfaces if needed
4. Add tests in `packages/lark/tests/`

#### 3. New Injected Scripts
1. Create `.ts` file in `src/scripts/`
2. Rollup will automatically include via glob pattern
3. Add corresponding menu item in `src/background.ts`
4. Update `MenuItemId` enum and message types

### Debugging Communication Issues

#### 1. Enable Comprehensive Logging
- Add detailed console.log statements with prefixes: `[Background]`, `[Content]`, `[WeChat Preview]`
- Log message payloads and response status
- Track pending request states in background script

#### 2. Verify Message Flow
1. Check injected script executes and posts messages
2. Verify content script receives and forwards messages
3. Confirm background script processes requests
4. Validate sidebar receives responses

#### 3. Timeout Handling
- Background script uses 20-second timeout for markdown requests
- Implement proper error handling for all async operations
- Clear pending requests on timeout to prevent memory leaks

### UI/UX Development Guidelines

#### 1. WeChat Preview Design
- Use full viewport height (100vh) for immersive experience
- Implement floating toolbar with backdrop blur effects
- Maintain responsive design for mobile compatibility
- Optimize padding/spacing for maximum reading area

#### 2. Copy Functionality
- Use DOM Range Selection API for WeChat compatibility
- Test with WeChat backend editor to ensure formatting preservation
- Implement visual feedback (success states, loading indicators)
- Handle both successful copy and fallback scenarios

### Quality Assurance Checklist

#### Before Each Feature Release
1. **Build Success**: `pnpm run build` completes without errors
2. **Type Safety**: `pnpm run type-check` passes
3. **Code Style**: `pnpm run lint` and `pnpm run format` pass
4. **Functionality Test**: Test in actual browser environment
5. **Cross-Context Communication**: Verify all message passing works
6. **Error Handling**: Test error scenarios and timeouts
7. **Color Consistency**: Verify no green elements remain in UI

#### Performance Considerations
- Minimize bundle size by avoiding unnecessary dependencies
- Use dynamic imports for large features when possible
- Optimize CSS for smooth animations and transitions
- Test memory usage in long-running extension scenarios

### Troubleshooting Common Issues

#### 1. "Module not found" errors
- Check file extensions (`.ts` vs `.js`)
- Verify import paths are correct
- Ensure rollup.config.js includes the file

#### 2. "Cannot access chrome.runtime" 
- Script is likely running in MAIN world instead of ISOLATED
- Use window.postMessage to communicate to content script

#### 3. UI scripts not loading
- Check module format (should be IIFE for HTML script tags)
- Verify rollup output configuration
- Test script loading in browser dev tools

#### 4. Communication timeouts
- Check network conditions for external API calls
- Verify message passing chain is complete
- Increase timeout values if processing is legitimately slow

---

## 🚨 Critical Development Workflow

### Pre-Development Checklist
Before starting any development task:

1. **📖 Review Documentation**: Always read [UI Design Guide](./docs/UI-DESIGN-GUIDE.md) and [Development Guide](./docs/DEVELOPMENT-GUIDE.md)
2. **🎨 Check Design System**: Verify color scheme, spacing, and component patterns
3. **⚙️ Understand Architecture**: Review message passing, file structure, and build system
4. **🔍 Learn from History**: Check this file's "Development Best Practices & Lessons Learned" section

### Post-Development Checklist
After completing any feature:

1. **🔧 Build Verification**: Run `pnpm run build` and verify success
2. **🎨 Design Consistency**: Ensure colors match primary scheme (`#d97759`)
3. **📱 Responsive Testing**: Test mobile and desktop layouts
4. **🔄 Communication Testing**: Verify all message passing works
5. **📚 Documentation Update**: Update guides if new patterns were established

**Remember: These documents capture hard-learned lessons. Following them prevents repeating past mistakes and ensures consistent, high-quality code.**

---
> Source: [geekjourneyx/feishu-md2wechat](https://github.com/geekjourneyx/feishu-md2wechat) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-05 -->
