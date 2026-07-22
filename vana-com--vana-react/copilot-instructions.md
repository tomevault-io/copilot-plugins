## vana-react

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is the `@opendatalabs/vana-react` package - a React component library for integrating with the Vana Network. It provides embeddable components for Vana Apps to facilitate data upload operations through iframe-based cross-origin communication.

## Core Architecture

### VanaAppUploadWidget

The main component that renders an iframe pointing to `app.vana.com/embed/upload`. Key architectural points:

- Uses postMessage API for secure cross-origin communication
- Handles bidirectional message flow between parent window and iframe
- Supports dynamic height resizing based on iframe content
- Strongly typed with TypeScript interfaces for props and theme

### Message Protocol

Communication flow between widget and Vana App:

1. **ready**: Iframe signals it's loaded → Widget sends config
2. **config**: Widget sends appId, schemaId, prompt, theme to iframe
3. **relay**: Bidirectional message forwarding
4. **auth**: User wallet authentication events
5. **complete**: Successful data upload
6. **error**: Error handling
7. **resize**: Dynamic height adjustment

### Future Hooks (Not Yet Implemented)

Planning to add these SDK-integrated hooks:

- `useDataContribution` - Complete data contribution flow
- `useFileUpload` - File uploads with schema validation
- `useTrustServer` - Server trust management
- `useUserFiles` - User data file management

## Development Commands

```bash
# Build & Development
npm run build                  # Build with tsup (outputs ESM + CJS)
npm run dev                    # Build with watch mode
npm run docs                   # Generate TypeDoc documentation
npm run docs:watch             # Generate docs with watch mode

# Testing
npm test                       # Run tests with vitest in watch mode
npm run test:coverage          # Run tests once with coverage report
npm test -- VanaAppUploadWidget.test.tsx  # Run specific test file

# Code Quality
npm run lint                   # Run eslint on src/
npm run lint:fix               # Auto-fix linting issues
npm run typecheck              # TypeScript type checking

# Local Testing
npm link                       # Create local package link
npm link @opendatalabs/vana-react  # Use in another project

# Publishing
npm version patch              # Bump patch version
npm version minor              # Bump minor version
npm version 0.1.0-alpha.1      # Set specific alpha version
npm publish --tag alpha        # Publish alpha to npm
npm publish                    # Publish stable to npm
```

## Build Configuration

### tsup Configuration

- Outputs both ESM and CJS formats with `"use client"` directive
- External dependencies: `react`, `react-dom`, `@opendatalabs/vana-sdk`, `wagmi`
- Target: ES2022
- Source maps and TypeScript declarations included
- No minification to aid debugging

### Testing Configuration

- **Vitest** with jsdom environment and React Testing Library
- Coverage thresholds: 80% lines/statements, 75% branches, 50% functions
- Test files co-located with components (`*.test.tsx`)

## Release Process

1. **Conventional Commits Required**: PR titles must follow format (`feat:`, `fix:`, `chore:`)
2. **Automated Release on main**: Semantic release handles versioning and npm publishing
3. **Alpha Testing**: Manual publish with `npm publish --tag alpha`
4. **GitHub Pages Docs**: Auto-deployed from main branch

## Key Implementation Details

### VanaAppUploadWidget Props

- `appId`: From Developer Portal (app.vana.com/build)
- `schemaId`: Optional override for app's default schema
- `prompt`: LLM inference prompt with `{{data}}` placeholder for trusted server
- `theme`: Strongly typed CSS custom properties (VanaAppUploadTheme interface)
- `iframeOrigin`: Defaults to `https://app.vana.com`

### Theme System

Uses CSS custom properties with TypeScript interface:

- Primary colors: `--primary`, `--primary-foreground`
- Layout colors: `--background`, `--card`, `--border`
- Text colors: `--foreground`, `--muted-foreground`
- Allows any additional `--*` properties via template literal type

### Testing Approach

- Test message passing between iframe and parent window
- Verify origin validation for security
- Mock postMessage events
- Test all callback props fire correctly

---
> Source: [vana-com/vana-react](https://github.com/vana-com/vana-react) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-22 -->
