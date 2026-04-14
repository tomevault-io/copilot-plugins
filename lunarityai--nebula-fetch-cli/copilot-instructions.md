## nebula-fetch-cli

> **nebula-fetch-cli** is a command-line interface (CLI) tool designed for downloading media from various platforms. It is built using TypeScript and the Bun runtime.

# Nebula Fetch CLI Context

## Project Overview

**nebula-fetch-cli** is a command-line interface (CLI) tool designed for downloading media from various platforms. It is built using TypeScript and the Bun runtime.

*   **Primary Function:** Currently supports downloading videos and extracting audio from YouTube.
*   **Key Libraries:**
    *   `commander`: For building the CLI interface.
    *   `chalk`: For styling terminal output.
    *   `@ybd-project/ytdl-core`: For handling YouTube media downloads.
    *   `bun`: Runtime and bundler.

## Architecture

The project follows a modular structure:

*   **Entry Point (`index.ts`):** Sets up the CLI program, defines commands (e.g., `youtube`), and handles argument parsing using `commander`.
*   **Commands (`commands/`):** Contains the implementation logic for specific commands.
    *   `youtube.ts`: Handles the logic for fetching video info and downloading streams.
*   **Utilities (`utils/`):** Helper functions and validators.
    *   `validators.ts`: Contains validation logic (e.g., checking for valid URLs).
*   **Configuration:**
    *   `tsconfig.json`: TypeScript configuration (Target: ESNext, Module Resolution: Bundler, Path Alias: `@/*`).
    *   `package.json`: Project metadata, scripts, and dependencies.

## Building and Running

The project utilizes `bun` for script execution and building.

### Scripts

*   **Development Mode:**
    ```bash
    bun run dev
    # or directly
    bun index.ts
    ```
*   **Build:**
    Compiles the TypeScript code into a single executable JavaScript file in the `dist` directory.
    ```bash
    bun run build
    ```
    *Output:* `./dist/index.js`

### CLI Usage (Development)

To test the CLI commands during development:

```bash
bun index.ts youtube <url> [options]
```

**Options:**
*   `-o, --output <path>`: Specify output path.
*   `-a, --audio`: Download audio only (mp3).
*   `-v, --verbose`: Show detailed debug information.

## Development Conventions

*   **Runtime:** The project is designed to run on **Bun**. Ensure Bun is installed.
*   **Path Aliases:** Use the `@/` alias to import files relative to the project root (configured in `tsconfig.json`).
*   **Styling:** Use `chalk` to provide visual feedback (e.g., green for success, red for errors, blue/cyan for info).
*   **Error Handling:** graceful error handling with informative messages using `console.error` and `process.exit(1)` for fatal errors.
*   **Type Safety:** Strict TypeScript mode is enabled. Ensure all types are properly defined.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/LUNARITYai)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/LUNARITYai)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
