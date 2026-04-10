## fast-check-frontend

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Global

In all interactions, be extremely concise and sacrifice grammar for the sake of concision.

## Project Overview

**fast-check-frontend** is a property-based testing library for UI components that generates and executes random user interaction sequences to verify component invariants. The name means "belt and suspenders" in French, representing redundant safety measures.

## Core Architecture

### Main Library (`src/createInteractionProperty.ts`)

The library provides a `createInteractionProperty` function that integrates fast-check property-based testing with Testing Library user events:

- **User Interaction Types**: Comprehensive set of interaction types (click, type, keyboard, hover, unhover, select, upload, clear, tab) with type-safe definitions
- **Arbitraries**: fast-check arbitraries generate realistic interaction sequences with configurable weights:
  - Clicks (30% weight), typing (25% weight), keyboard actions (10% weight)
  - Selectors include data-testid attributes, ARIA roles, and element types
  - Text content includes edge cases: empty strings, Unicode, XSS attempts, SQL injection patterns
- **Execution Engine**: `executeInteraction` and `executeInteractionSequence` functions gracefully handle missing DOM elements (log but don't fail)
- **Property Testing Integration**: `createInteractionProperty` accepts:
  - `renderComponent`: Function that returns a fresh HTMLElement
  - `invariants`: Array of sync/async predicates to check after interaction sequences
  - `options`: Configure sequence length (min/max)

### Example Implementation (`examples/react/`)

A React signup form (`SignupForm.tsx`) demonstrates the library usage:

- Complex form with validation, modals, error boundaries
- Multiple interaction targets: inputs, buttons, checkboxes, select dropdowns
- Property test (`SignupForm.test.tsx`) verifies three invariants under random interactions:
  1. No JavaScript errors (no error boundary rendered)
  2. Required fields maintain aria-required attribute
  3. Never more than one modal open simultaneously

## Development Commands

### React Example
```bash
cd examples/react
pnpm install
pnpm dev          # Start Vite dev server
pnpm build        # TypeScript compile + Vite build
pnpm lint         # Run ESLint
pnpm test         # Run Vitest tests
```

### Running Tests
```bash
# From examples/react directory
pnpm test                    # Run all tests in watch mode
pnpm test -- --run          # Run tests once (CI mode)
pnpm test -- SignupForm     # Run specific test file
```

## Technical Details

- **Package Manager**: pnpm with workspaces (root + examples/react)
- **Test Stack**: Vitest with jsdom environment, Testing Library React, fast-check
- **Build Tools**: Vite (using rolldown-vite experimental bundler), TypeScript 5.9
- **React**: React 19 with React Compiler enabled

## Key Patterns

1. **Interaction Testing Philosophy**: Instead of testing specific user paths, generate random interaction sequences and verify invariants always hold
2. **Graceful Degradation**: Interaction execution catches errors and continues - selectors might not match, elements might not exist, that's expected
3. **Invariant-Based Validation**: Define what should NEVER happen (crash, accessibility violations, invalid state) rather than prescriptive behavior
4. **Weighted Arbitraries**: Common interactions (click, type) have higher probability than rare ones (upload, unhover)

## Project Structure

```
/src/createInteractionProperty.ts   # Main library code
/examples/react/                     # React example implementation
  src/SignupForm.tsx                # Complex form component
  src/SignupForm.test.tsx           # Property test using the library
  src/App.tsx                       # Demo app entry
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/mdubourg001)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/mdubourg001)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
