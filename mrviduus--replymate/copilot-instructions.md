## replymate

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

ReplyMate is a privacy-focused Chrome extension (Manifest V3) that provides AI-powered LinkedIn reply suggestions. It runs entirely locally using WebLLM for AI inference without sending data to external servers.

## Essential Commands

### Development
- `npm run dev` - Start development with hot reload (Parcel)
- `npm run build` - Production build with webextension config
- `npm run clean` - Clean build artifacts

### Testing
- `npm test` - Run full test suite (132+ tests)
- `npm run test:watch` - Run tests in watch mode
- `npm run test:coverage` - Generate coverage report
- To run a single test: `npm test -- path/to/test.spec.ts`

### Code Quality
- `npm run lint` - Check code with ESLint (must pass with 0 warnings)
- `npm run lint:fix` - Auto-fix linting issues
- `npm run type-check` - TypeScript type checking (strict mode)
- `npm run format` - Format code with Prettier

### Building & Packaging
- `npm run build:script` - Full build pipeline with quality checks (./scripts/build.sh)
- `npm run package` - Create distribution packages
- `npm run zip` - Build and package in one command
- `npm run validate-manifest` - Validate manifest.json structure

## Architecture

### Core Components

1. **Background Service Worker (src/background.ts)**
   - Manages WebLLM AI engine lifecycle
   - Handles model selection with smart fallbacks
   - Processes inference requests from content scripts
   - Contains LinkedIn-specific prompt engineering

2. **LinkedIn Content Script (src/linkedin-content.ts)**
   - Injects "Generate Reply" buttons into LinkedIn UI
   - Extracts post/comment content for AI processing
   - Manages reply generation flow and UI updates
   - Handles compliance warnings and error states

3. **Popup Interface (src/popup.ts, popup.html)**
   - Main extension control panel
   - Model selection and management
   - Settings and customization for LinkedIn reply generation

### AI Configuration
- **Library**: @mlc-ai/web-llm v0.2.79
- **Model Tiers**: Professional (Llama-3.2), Balanced (Qwen2.5), Fast (Gemma-2, Phi-3.5)
- **Smart Selection**: Automatic model selection based on device capabilities
- **Local Processing**: All AI inference happens on-device for privacy

### Extension Architecture
- **Manifest V3**: Modern Chrome extension architecture
- **Content Security Policy**: Strict CSP for security
- **Message Passing**: Chrome runtime messaging between components
- **Storage**: Chrome storage API for settings persistence

## Development Guidelines

### TypeScript Configuration
- Target: ES2020
- Strict mode enabled
- Module resolution: Node
- Source maps enabled for debugging

### Testing Approach
- Framework: Jest with ts-jest and jsdom
- Setup: tests/setup.ts provides Chrome API mocks
- Coverage: Collect from src/**/*.{ts,tsx}
- Run specific tests: `npm test -- --testNamePattern="pattern"`

### Build Process (scripts/build.sh)
1. Validates dependencies
2. Runs TypeScript type checking
3. Executes ESLint checks
4. Runs test suite
5. Compiles TypeScript
6. Copies static assets to dist/
7. Validates manifest.json
8. Reports build size

### Code Quality Requirements
- TypeScript strict mode must pass
- ESLint must report 0 warnings
- All tests must pass before build
- Prettier formatting must be applied

## Key Directories

- `src/` - Main source code (TypeScript)
- `tests/` - Test files with Chrome API mocks
- `dist/` - Build output (git-ignored)
- `scripts/` - Build and packaging automation
- `docs/` - Architecture and troubleshooting guides
- `icons/` - Extension icons (multiple sizes)

## Working with LinkedIn Integration

The extension specifically targets LinkedIn's DOM structure:
- Post detection: `div[data-id]` containers
- Comment forms: `div.comments-comment-box`
- Reply textareas: `div[contenteditable="true"]`
- Button injection points vary by post type

When modifying LinkedIn integration:
1. Test on different post types (articles, posts, comments)
2. Handle dynamic content loading
3. Respect LinkedIn's rate limits
4. Test with various LinkedIn UI themes

## Chrome Extension Specifics

### Permissions Required
- storage: Settings persistence
- activeTab: Content script injection
- Host permissions: https://www.linkedin.com/*

### Message Types
- `generateReply`: Request AI reply generation
- `getModelsInfo`: Fetch available models
- `testModel`: Test model functionality
- `resetModel`: Clear current model

### Storage Keys
- `selectedModel`: Current AI model
- `customPrompt`: User's custom prompt
- `settings`: Extension configuration

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/mrviduus) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
