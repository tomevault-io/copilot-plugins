## morphie

> **Morphie** is a CLI tool designed to port open source projects from one programming language to another using local Large Language Models (LLMs) via Ollama. It aims to perform 1:1 code translation while preserving directory structure, functionality, and adopting the target language's idioms.

# Morphie - Project Context

## Project Overview

**Morphie** is a CLI tool designed to port open source projects from one programming language to another using local Large Language Models (LLMs) via Ollama. It aims to perform 1:1 code translation while preserving directory structure, functionality, and adopting the target language's idioms.

## Architecture & Codebase Structure

The project is a TypeScript-based CLI application built with Node.js.

### Key Directories

- **`src/`**: Source code root.
    - **`cli.ts`**: Main entry point for the CLI. Sets up `commander` and command definitions.
    - **`index.ts`**: Library exports (programmatic API).
    - **`commands/`**: Implementation of CLI commands.
        - `port.ts`: Handles the main porting workflow (`morphie port`).
        - `analyze.ts`: Handles project analysis (`morphie analyze`).
        - `models.ts`: Lists available Ollama models (`morphie models`).
    - **`core/`**: Core business logic.
        - `analyzer.ts`: Scans source directories, identifies languages, and maps files.
        - `porting-engine.ts`: Orchestrates the porting process, constructs prompts, and manages LLM interactions.
    - **`llm/`**: LLM integration layer.
        - `ollama.ts`: Client for the Ollama REST API.
    - **`utils/`**: Helper utilities.
        - `languages.ts`: Definitions for supported languages, extensions, and mapping logic.
        - `filesystem.ts`: File I/O operations.
        - `project-config.ts`: Configuration management.

### Tech Stack

- **Runtime**: Node.js (>=18.0.0)
- **Language**: TypeScript
- **CLI Framework**: `commander`
- **Utilities**: `chalk` (styling), `ora` (spinners), `glob` (file matching)
- **Testing**: `vitest`
- **Linting**: `eslint`

## Development Workflow

### Setup

```bash
npm install
```

### Build & Run

*   **Development Mode (no build required):**
    ```bash
    npm run dev -- <command> <options>
    # Example: npm run dev -- port ./src ./out -f typescript -t python
    ```

*   **Build:**
    ```bash
    npm run build
    # Compiles TypeScript to ./dist
    ```

*   **Run Production Build:**
    ```bash
    npm start -- <command>
    # or
    node dist/cli.js <command>
    ```

### Testing

*   **Run all tests:**
    ```bash
    npm test
    ```

*   **Watch mode:**
    ```bash
    npm run test:watch
    ```

### Code Quality

*   **Linting:**
    ```bash
    npm run lint
    ```

*   **Fix Lint Issues:**
    ```bash
    npm run lint:fix
    ```

## Usage

Morphie requires an [Ollama](https://ollama.ai) instance running locally (default: `http://localhost:11434`).

### Common Commands

*   **Port a project:**
    ```bash
    morphie port <source> <target> -f <from-lang> -t <to-lang>
    ```
    *Options:*
    *   `-m, --model <name>`: Specify LLM model (default: `codellama`)
    *   `--dry-run`: Analyze without generating files

*   **Analyze source:**
    ```bash
    morphie analyze <source>
    ```

*   **List models:**
    ```bash
    morphie models
    ```

## Conventions

*   **Language Support**: New supported languages should be added to `src/utils/languages.ts`.
*   **LLM Interaction**: All LLM calls go through `src/llm/ollama.ts`.
*   **Prompting**: Prompts are constructed in `src/core/porting-engine.ts` and should include context about both source and target languages.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/dotkebi)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/dotkebi)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
