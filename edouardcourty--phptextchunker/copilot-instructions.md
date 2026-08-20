## phptextchunker

> **php-text-chunker** is a standalone PHP library for splitting text (from files or strings) into semantically meaningful chunks, using pluggable strategies and a composable post-processing pipeline.

# AGENTS.md - Coding Guidelines for AI Agents

## 🎯 Core Concept

**php-text-chunker** is a standalone PHP library for splitting text (from files or strings) into semantically meaningful chunks, using pluggable strategies and a composable post-processing pipeline.

### Problem Solved

When building RAG (Retrieval-Augmented Generation) pipelines, vector search systems, or any NLP application, raw text must be split into smaller chunks before embedding or processing. Different content types (articles, dialogues, code, etc.) require different splitting strategies.

### Solution

A strategy-based chunker with a streaming-first design:
- Pluggable **chunking strategies** for different content types
- A composable **post-processor pipeline** applied after chunking
- Memory-efficient **Generator-based streaming** (no full file load)
- Works with both **files** and **raw strings**

---

## 🏗️ Architecture

### Overview

```
TextChunker (entry point)
  ├── setFile(path) / setText(string)
  ├── withReader(ReaderInterface)
  ├── withMetadata(array)
  ├── withPostProcessor(ChunkPostProcessorInterface)
  └── chunk(ChunkingStrategyInterface): Generator<Chunk>
           │
           ├── ReaderInterface (streams raw data from source)
           ├── Strategy (splits data into Chunks)
           └── PostProcessors (pipeline applied in order)
```

### Main Components

| Component                     | Location                    | Role                                               |
|-------------------------------|-----------------------------|----------------------------------------------------|
| `TextChunker`                 | `src/TextChunker.php`       | Entry point. Orchestrates chunking.                |
| `Chunk`                       | `src/ValueObject/Chunk.php` | Immutable value object: text + position + metadata |
| `ChunkingStrategyInterface`   | `src/Contract/`             | Contract for splitting strategies                  |
| `ChunkPostProcessorInterface` | `src/Contract/`             | Contract for post-processors                       |
| `ReaderInterface`             | `src/Contract/`             | Contract for file/stream readers                   |
| `LocalFileReader`             | `src/Reader/`               | Default reader using native PHP file functions     |
| Strategies                    | `src/Strategy/`             | 8 built-in splitting strategies                    |
| Post-processors               | `src/PostProcessor/`        | 8 built-in post-processing transforms              |

---

## 🚀 Typical Use Cases

- Preparing document chunks for vector embedding (RAG pipelines)
- Splitting large text files for batch NLP processing
- Preprocessing dialogue transcripts into conversation segments
- Enforcing token limits before sending text to LLM APIs

---

## 💡 Design Patterns Used

- **Strategy Pattern** — Swap chunking logic at runtime
- **Pipeline / Decorator** — Post-processors wrap Generators in sequence
- **Generator Streaming** — Memory-efficient; never loads full file into memory
- **Immutable Value Object** — `Chunk` is `readonly`; `withMetadata()` returns a new instance
- **Fluent Interface** — `TextChunker` methods are chainable

---

## Project breakdown

### Strategies (`src/Strategy/`)

| Class | Splits on |
|---|---|
| `ParagraphChunkingStrategy` | Double newlines (`\n\n`) |
| `SentenceChunkingStrategy` | Sentence-ending punctuation (`[.!?]`) |
| `FixedSizeChunkingStrategy` | Fixed character count (configurable) |
| `MarkdownChunkingStrategy` | Markdown headers (`#` to `######`) |
| `WordCountChunkingStrategy` | Fixed word count |
| `RegexChunkingStrategy` | Configurable regex pattern |
| `LineChunkingStrategy` | N consecutive lines per chunk |
| `RecursiveChunkingStrategy` | Cascade of strategies with a max chunk size |

### Post-Processors (`src/PostProcessor/`)

| Class | Purpose |
|---|---|
| `OverlappingChunkPostProcessor` | Prepends tail of previous chunk for context continuity |
| `TokenLimitPostProcessor` | Splits oversized chunks to respect token budget |
| `MetadataEnricherPostProcessor` | Adds word count, char count, index, total, source |
| `ChunkFilterPostProcessor` | Removes empty or too-short chunks |
| `ChunkMergerPostProcessor` | Merges consecutive small chunks up to a minimum size |
| `TextNormalizationPostProcessor` | Cleans whitespace, control chars, trims lines |
| `DeduplicationPostProcessor` | Removes duplicate chunks by content hash (md5) |
| `RegexReplacePostProcessor` | Applies regex substitutions on each chunk's text |

**IMPORTANT**: This section should evolve with the project. When a new feature is created, updated or removed, this section should too.

## 🧪 Testing

Tests are located in `tests/Unit/`. Each strategy and post-processor has its own test class.

```
tests/Unit/
├── TextChunkerTest.php
├── Strategy/
└── PostProcessor/
```

Run tests: `composer test`

---

## Remarks & Guidelines

### General

- NEVER commit or push the git repository.
- When unsure about something, you MUST ask the user for clarification.
- Always choose robust solutions over hacky fixes.
- ALWAYS write tests for new components.
- Do NOT write type documentation unless explicitly asked.
- Once a feature is complete, update `README.md` and `AGENTS.md` accordingly.

### Adding a new Strategy

1. Create `src/Strategy/MyStrategy.php` implementing `ChunkingStrategyInterface`
2. Implement `process(string $data, bool $isEnd): \Generator` and `reset(): void`
3. Add metadata key `strategy` to each yielded `Chunk`
4. Add unit tests in `tests/Unit/Strategy/`

### Adding a new Post-Processor

1. Create `src/PostProcessor/MyProcessor.php` implementing `ChunkPostProcessorInterface`
2. Implement `process(\Generator $chunks, string $source = ''): \Generator`
3. Stream chunks — avoid buffering unless strictly necessary
4. Add unit tests in `tests/Unit/PostProcessor/`

### Adding a new Reader

1. Create `src/Reader/MyReader.php` implementing `ReaderInterface`
2. Implement `readChunks(string $path, int $bufferSize): \Generator` — yield `string` data chunks
3. Throw `\RuntimeException` (or a subtype) if the source cannot be opened or read
4. Add unit tests in `tests/Unit/Reader/`

## 📚 References

- **Source code**: `/src`
- **Tests**: `/tests`
- **README**: User documentation

---
> Source: [EdouardCourty/PHPTextChunker](https://github.com/EdouardCourty/PHPTextChunker) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-20 -->
