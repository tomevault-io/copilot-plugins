## elixir-ai-sdk

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This project is an Elixir port of the Vercel AI SDK, designed to help developers build AI-powered applications using Elixir and Phoenix. The codebase follows a test-driven development approach, implementing features incrementally based on the tests in the original Vercel AI SDK.

## Development Commands

- Run all tests: `mix test`
- Run specific test file: `mix test test/path/to/file.exs`
- Run specific test line: `mix test test/path/to/file.exs:123`
- Format code: `mix format`
- Check code compilation: `mix compile`

## Testing Strategy

- When adding tests, make a plan, but then add the tests one-by-one, running them each time, and only proceeding to the next one when the current one passes.
- Tests are based on the corresponding JavaScript tests in the Vercel AI SDK, but adapted for Elixir's idioms and patterns.
- The TEST_PLAN.md file tracks the implementation progress of tests ported from the Vercel AI SDK.

## Code Architecture

- The codebase follows a modular structure with the core AI functionality in the `AI` module.
- `AI.Core` modules handle the implementation details while the main `AI` module provides the public API.
- `MockLanguageModel` provides test implementations of language models with predefined behaviors.
- The architecture is designed to accommodate multiple AI providers (OpenAI, Anthropic, etc.) while presenting a unified API.

## Key Components

- `AI` - The main module that exposes the public API
- `AI.Core.GenerateText` - Core implementation of text generation functionality
- `AI.Core.MockLanguageModel` - Mock implementation for testing

## Implementation Patterns

- Use pattern matching and guard clauses for handling different response formats
- Extract helper functions for common operations (e.g., `extract_reasoning`, `extract_files`)
- Follow Elixir idioms like returning tagged tuples (e.g., `{:ok, result}` or `{:error, reason}`)

---
> Source: [elepedus/elixir-ai-sdk](https://github.com/elepedus/elixir-ai-sdk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
