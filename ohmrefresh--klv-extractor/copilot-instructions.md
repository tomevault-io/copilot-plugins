## klv-extractor

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a React-based KLV (Key-Length-Value) data extraction and processing suite for Paymentology transaction data. The application provides a complete toolkit for parsing, building, and batch processing KLV formatted data.

## Development Commands

- `npm start` - Runs the app in development mode with Vite (http://localhost:3000 or next available port)
- `npm run dev` - Alternative command to start development server
- `npm run build` - Builds the app for production to the `build` folder using TypeScript and Vite
- `npm run preview` - Preview the production build locally
- `npm test` - Launches Vitest test runner in interactive watch mode
- `npm run test:coverage` - Runs all tests with coverage report
- `npm run test:ci` - Runs tests in CI mode with coverage (no watch)
- `npm run test:ui` - Opens Vitest UI for interactive test running

## Core Architecture

### KLV Parser Engine (`src/utils/KLVParser.ts`)
The heart of the application is the KLVParser utility which:
- Contains complete KLV definitions for 100+ Paymentology fields (keys 002-999)
- Parses KLV strings with format validation and error handling
- Exports data to JSON, CSV, and table formats
- Builds KLV strings from component entries
- Validates KLV data structure and integrity

### Component Architecture
The application uses a tab-based interface with four main sections:

**Main App (`src/App.jsx`)**
- Manages application state (activeTab, klvInput, searchTerm, history)
- Handles file uploads and history management
- Coordinates between different processing modes

**Core Components:**
- `FileUpload` - Handles file input for KLV data
- `ExportPanel` - Provides export functionality to various formats
- `Statistics` - Shows parsing statistics and data insights
- `KLVBuilder` - Interactive form for constructing KLV strings
- `BatchProcessor` - Processes multiple KLV entries simultaneously

### Data Flow
1. Input: KLV strings via manual input, file upload, or builder
2. Processing: KLVParser validates and parses data into structured format
3. Display: Results shown with search/filter capabilities
4. Export: Data can be exported to multiple formats
5. History: Successful parses saved to processing history

### Key Features
- **Real-time parsing** with immediate error feedback
- **Search and filter** across parsed results
- **Batch processing** for multiple KLV entries
- **Interactive builder** for creating KLV strings
- **Export capabilities** (JSON, CSV, table format)
- **Processing history** with load/copy functionality
- **Sample data** for testing and demonstration

## Technology Stack

- **React 18** with functional components and hooks
- **TypeScript** for type safety and better development experience
- **Vite** - Modern build tool and development server
- **Vitest** - Fast unit test framework with Jest-compatible API
- **Tailwind CSS** for styling (included as dev dependency)
- **Lucide React** for icons

## KLV Data Format

The application processes Key-Length-Value data with the structure:
- 3-digit key (002-999)
- 2-digit length field (hexadecimal)
- Variable-length value field
- Example: `00206AB48DE` = Key=002, Length=06, Value=AB48DE

## Development Notes

- Components follow React functional patterns with hooks and TypeScript interfaces
- State management is handled at the App level and passed down via typed props
- Error handling is built into the parsing engine with proper type safety
- The application uses TypeScript for compile-time type checking and better IDE support
- All interfaces and types are properly defined for KLV data structures
- The application is designed for defensive security (data parsing and analysis only)
- No server-side dependencies - pure client-side processing

## TypeScript Interfaces

Key interfaces defined in the codebase:
- `KLVEntry` - Represents a parsed KLV entry with key, length, value, position, and name
- `KLVParseResult` - Contains parsing results and any errors
- `KLVValidationResult` - Validation results for KLV data
- `KLVBuildEntry` - Entry format for building KLV strings

## Testing

The application includes comprehensive unit and integration tests:

### Test Structure
- **Unit tests** for KLV Parser utility (`src/tests/utils/KLVParser.test.ts`)
- **Component tests** for React components (`src/tests/components/*.test.tsx`)
- **Integration tests** for full application workflows (`src/tests/integration/*.test.tsx`)
- **Test utilities** for mocking and shared test helpers (`src/tests/helpers/testUtils.ts`)

### Test Coverage
Tests cover:
- KLV parsing with valid and invalid data
- Error handling and edge cases
- Component rendering and user interactions
- File upload functionality
- Export operations
- Search and filtering
- Tab navigation and state management
- Clipboard operations
- History management

### Running Tests
```bash
npm test                 # Interactive test runner
npm run test:coverage   # Tests with coverage report
npm run test:ci         # CI-friendly test run
```

### Test Dependencies
- Vitest (test framework with Jest-compatible API)
- React Testing Library (component testing)
- @testing-library/jest-dom (DOM assertions)
- @testing-library/user-event (user interaction simulation)
- @vitest/coverage-v8 (code coverage)

---
> Source: [ohmrefresh/klv-extractor](https://github.com/ohmrefresh/klv-extractor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-26 -->
