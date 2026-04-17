## ucu-m2-fp-wikistream

> This document provides foundational mandates and guidelines for Gemini CLI when working on the Wikipedia Real-Time Stream Analyzer project. These instructions take precedence over general defaults.

# GEMINI Instructions

This document provides foundational mandates and guidelines for Gemini CLI when working on the Wikipedia Real-Time Stream Analyzer project. These instructions take precedence over general defaults.

## 📖 Project Overview
This is a Functional Reactive Programming (FRP) implementation for analyzing Wikipedia/Wikimedia real-time change streams. It follows a specific "Professor's Pattern" using the `aiostream` library and custom Monad implementations for type safety and side-effect isolation.

## 🏗 Architectural Mandates

### 1. Functional Reactive Programming (FRP)
- **Library**: Use `aiostream>=0.5.2`.
- **Operators**: All streaming logic MUST use `@operator` and `@pipable_operator` decorators from `aiostream`.
- **Custom Operators**:
  - `map_maybe`: Use for applying functions that return a `Maybe` monad and filtering out `Nothing`.
  - `parse_stream`: Use for transforming raw dictionaries into `ParsedEvent` objects in a declarative pipeline. Each `ParsedEvent` includes `old_rev`/`new_rev` revision IDs required for diff fetching.
  - `filter_edit_type_stream`: Use for domain-specific filtering of edit types.
- **Composition**: Chain operators using the pipe `|` operator for declarative pipelines.
- **Rate Limiting**: Always include `aiostream.pipe.delay(0.1)` (100ms) in streaming pipelines to prevent overwhelming the system and respect the Wikimedia SSE endpoint.

### 2. Monad-Driven Development
Located in `functional_utils.py`, these monads MUST be used for consistency:
- **Maybe Monad**: Use for handling optional values and parsing potentially malformed JSON from the stream.
- **Either Monad**: Use for type-safe error handling in query APIs and business logic (prefer `Left(error)`/`Right(result)` over raising exceptions).
- **IO Monad**: Use to encapsulate side-effects (e.g., printing to console, writing files, updating in-memory state). Side-effects should only be executed by calling `.run()`.

### 3. Immutability & State Management
- **Pure Functional State**: Use `StatisticsStore` for state accumulation within `aiostream.pipe.accumulate`.
- **Functional Updates**: Every state update MUST return a new state instance rather than mutating the existing one. Use `store.update(event)` for event ingestion and `store.add_word_corrections(pairs)` for word correction data — both return new `StatisticsStore` instances.
- **Thread Safety**: Use `asyncio.Lock` when updating shared state within the `IO` monad execution. For pure functional pipelines, state is managed by the stream itself.

### 4. Word Correction Analysis (`word_mistake_analyzer.py`)
- All diff parsing and word alignment logic lives in `word_mistake_analyzer.py` — the only async I/O function is `fetch_revision_diff`; everything else is pure.
- Use `extract_corrections(diff_html)` to get `List[Tuple[str, str]]` correction pairs from an HTML diff.
- Use `analyze_event_corrections(old_rev, new_rev)` as the full async pipeline (returns `Either`).
- Persist results via `store.add_word_corrections(pairs)` — **do not** embed word data in `ParsedEvent`.
- Query via `get_most_mistaken_words(store)` from `query_api.py`.

## 🧪 Engineering Standards

- **Language**: Python 3.10+ (utilizing modern async/await syntax).
- **Testing**: `pytest` is the primary test runner. All new features or bug fixes MUST include corresponding tests in `test_functional.py`.
- **Code Style**: Adhere to existing patterns in the codebase. Ensure clear separation between pure functions and side-effect-heavy IO operations.
- **Documentation**: Maintain the detailed implementation notes in `README.md` for any major architectural changes.

## 🛠 Development Workflow

1. **Research & Reproduction**:
   - Before fixing a bug, reproduce it with a new test case in `test_functional.py`.
   - Study `functional_utils.py` before extending Monad capabilities.

2. **Execution**:
   - For UI/Visualization changes, verify the output in the `visualizations/` directory.
   - For streaming changes, use `aiostream.pipe.take(N)` to limit event counts during testing.

3. **Validation**:
   - Run `pytest test_functional.py -v` to ensure no regressions in Monad behavior or event processing.
   - Run `python main.py` to verify the pure functional `aiostream` pipeline integrity.

## 🚨 Safety & Security
- **Data Privacy**: Although the Wikipedia EventStream is public, avoid logging or storing personal identifying information (PII) beyond what is necessary for statistics.
- **Resource Management**: Always use `async with pipeline.stream() as streamer` to ensure proper resource cleanup of SSE connections.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/strateg17) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
