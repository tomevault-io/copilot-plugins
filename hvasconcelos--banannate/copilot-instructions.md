## use-bun-instead-of-node-vite-npm-pnpm

> Use Bun instead of Node.js, npm, pnpm, or vite.


# banannate - Development Guide

CLI tool for generating images using Google's Imagen API (Gemini Image Models) via Bun.

## Commands

- **Run CLI**: `bun index.ts --prompt "..." --output result.png`
- **Install**: `bun install`
- **Build (Local)**: `bun build --compile --minify --sourcemap ./index.ts --outfile banannate`
- **Build (Cross-platform)**: See README for specific `--target` flags (e.g., `bun-linux-x64`, `bun-darwin-arm64`).
- **Logo**: `bun run logo`

## Code Style & Conventions

- **Runtime**: Always use Bun. Avoid Node.js standard libraries if a Bun equivalent exists.
- **TypeScript**: Strictly typed. Use interfaces for API requests/responses and configuration.
- **Bun APIs**:
  - `Bun.file(path)` for file reading/existence checks.
  - `Bun.write(path, data)` for writing output images.
  - Native `fetch` for Google Gemini API calls.
  - `Bun.argv` for CLI argument access.
- **Error Handling**: Graceful error messages for API failures (401, 429, 400, 500) and invalid CLI arguments.
- **UI**: Use `LoadingSpinner` (in `index.ts`) for progress feedback during long-running API calls.
- **Commits**: Follow [Conventional Commits](https://www.conventionalcommits.org/).

## Supported Models

The tool supports the following Gemini image generation models (defined in `SUPPORTED_MODELS` constant):

- `gemini-2.5-flash-image` - Fast, efficient image generation
- `gemini-3-pro-image-preview` - High-quality image generation (default)

**Model Selection**:
- Default model: `gemini-3-pro-image-preview`
- Override via `--model` CLI flag
- Unsupported model names trigger warnings but are still passed to the API

## Project Structure

- `index.ts`: CLI entry point, argument parsing, API integration, and main workflow.
- `logo.ts`: ASCII art and terminal color constants.
- `CONTRIBUTING.md`: Detailed contribution workflow.
- `README.md`: Usage documentation and build instructions.

---
> Source: [hvasconcelos/banannate](https://github.com/hvasconcelos/banannate) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-25 -->
