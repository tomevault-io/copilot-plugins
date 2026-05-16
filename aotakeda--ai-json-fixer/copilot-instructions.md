## ai-json-fixer

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is the ai-json-fixer project, a simple JSON parser specifically designed to handle malformed JSON from Large Language Models (LLMs). The project uses TypeScript with comprehensive test coverage and follows Test-Driven Development principles.

## Commands

### Development

- `npm run dev` - Run development server with watch mode
- `npm run build` - Build TypeScript to JavaScript
- `npm run typecheck` - Run TypeScript type checking

### Testing

- `npm test` - Run all tests (77 passing out of 86 total)
- `npm run test:watch` - Run tests in watch mode
- `npm run test:coverage` - Run tests with coverage report

### Code Quality

- `npm run lint` - Run ESLint
- `npm run format` - Format code with Prettier

### Examples

- `npx tsx examples/basic-usage.ts` - Run usage examples

## Architecture

The project follows a modular, TDD-based architecture:

### Core Structure

- `src/fixes/` - Individual fix modules (4 completed: markdown-extraction, trailing-content-removal, quote-fixing, missing-comma-detection)
- `src/parsers/` - Main parser implementation (`LLMJSONParser`)
- `src/types/` - TypeScript interfaces and types
- `src/index.ts` - Main exports
- `test/unit/` - Unit tests for individual modules (68 tests, all passing)
- `test/integration/` - Integration tests (18 tests, 9 passing)
- `examples/` - Usage examples

### Fix System

Each fix is implemented as a separate, testable module with:

- Clear function interface exported from `src/fixes/`
- Comprehensive unit tests with 100% pass rate
- Ability to be chained together in the main parser
- Error recovery and graceful degradation

### Completed Features (Phase 1 MVP - Production Ready)

- **Markdown Block Extraction**: Extracts JSON from ```json code blocks and inline `code` - handles GPT/Claude output
- **Trailing Content Removal**: Removes explanatory text after valid JSON structures using bracket matching
- **Quote Fixing**: Fixes unescaped quotes inside JSON strings with context awareness
- **Missing Comma Detection**: Adds missing commas between elements (works on multi-line JSON)
- **Main Parser Class**: `LLMJSONParser` with configurable options, detailed results, and error tracking

### Current Status

- **77/86 tests passing** (90% success rate)
- **All unit tests passing** (68/68)
- **Core functionality working** for real LLM outputs
- **Zero dependencies** - pure TypeScript implementation
- **Full TypeScript support** with strict configuration

## Usage Patterns

### Basic Usage

```typescript
import { LLMJSONParser } from 'ai-json-fixer';
const parser = new LLMJSONParser();
const result = parser.parse(llmOutput);
```

### Common Fix Scenarios

1. **Markdown extraction**: Handles ChatGPT/Claude responses with ```json blocks
2. **Quote issues**: Fixes `{"msg": "He said "hello""}` → `{msg: 'He said "hello"'}`
3. **Trailing text**: Fixes `{"status": "ok"} Success!` → `{status: "ok"}`
4. **Missing commas**: Fixes multi-line JSON with missing separators

### Configuration Options

- `mode`: 'strict' | 'standard' | 'aggressive'
- `stripMarkdown`, `trimTrailing`, `fixQuotes`, `addMissingCommas`: boolean toggles
- `trackFixes`: Get detailed information about applied fixes
- `throwOnError`: Control error handling behavior

## Testing Strategy

The project uses Test-Driven Development (TDD):

1. **Write failing tests first** for each feature
2. **Implement minimal code** to pass tests
3. **Refactor** for clarity and performance
4. **Maintain high test coverage** (currently 90% overall)

### Test Organization

- Unit tests: One file per fix module, comprehensive edge case coverage
- Integration tests: Real-world LLM output scenarios
- All individual fixes have 100% test pass rate

## Known Limitations

- Missing comma detection works best with multi-line JSON (single-line has limitations)
- Some complex nested scenarios in integration tests need refinement
- Streaming support not yet implemented (planned for Phase 2)

## Future Development

Next phases include streaming support, advanced parsing features, and enhanced developer experience tools.

---
> Source: [aotakeda/ai-json-fixer](https://github.com/aotakeda/ai-json-fixer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-13 -->
