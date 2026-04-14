## docs-kit-mcp

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**docs-kit** is an intelligent documentation agent built on MCP (Model Context Protocol). It performs semantic change detection via AST analysis (tree-sitter) combined with Git diffs to automatically update documentation when code changes. The full specification is in `start.md` (written in Portuguese).

## Tech Stack (Planned)

- TypeScript / Node.js
- tree-sitter for multi-language AST parsing
- SQLite for index storage
- Jest for testing
- MCP server protocol for VS Code/Copilot integration

## Architecture

The system is organized in layers:

- **Analysis Layer**: AST-based indexer + Git semantic change analyzer
- **Knowledge Layer**: Symbol relationship graph + RAG/vector DB for semantic search
- **Documentation Layer**: Symbol-to-doc registry (frontmatter mapping), section-level doc updater (non-destructive), executable doc validation
- **Governance Layer**: Pattern detection, architecture rule enforcement (Arch Guard), Docs-Guard CI gate, dead code/orphan doc reaper
- **Business Layer**: Ticket-to-code-to-docs traceability, requirements matrix, natural language generation
- **Interface Layer**: MCP server, CLI tools (audit, impactAnalysis, createOnboarding), OpenAPI/GraphQL sync

Central data model revolves around `CodeSymbol` with kinds: class, method, function, interface, dto, entity, event, listener.

## Running the MCP Server

```bash
node dist/server.js &
```

## Test Structure

Tests are colocated with source code using `__tests__` folders inside each directory:

```
src/
├── analysis/
│   ├── __tests__/
│   │   └── analyzer.test.ts
│   └── analyzer.ts
├── cli/
│   ├── usecases/
│   │   ├── __tests__/
│   │   │   └── analyzePatterns.usecase.test.ts
│   │   └── analyzePatterns.usecase.ts
│   └── ...
└── ...
```

**Conventions:**
- Test files follow the pattern `<filename>.test.ts`
- Each `__tests__` folder is placed alongside the code it tests
- Use Jest as the test runner

## Key Design Decisions

- Documentation updates only modify relevant sections; never create new files or destroy existing content
- AST-first approach: semantic diffs over text diffs
- Docs-Guard: CI builds fail if semantic changes lack corresponding doc updates
- Use `z.uuid()` (not the deprecated `z.string().uuid()`)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/marcuspmd)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/marcuspmd)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
