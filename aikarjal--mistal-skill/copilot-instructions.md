## mistal-skill

> Convert local PDF and image files into markdown using Mistral OCR. Use when an agent needs reliable document text extraction for downstream analysis, summarization, chunking, indexing, or QA workflows where the source input is a PDF/image file and the output must be markdown.


# Mistral OCR Markdown

## Overview

Use the bundled TypeScript CLI to convert one local PDF/image file into one markdown file via Mistral OCR.

## Workflow

1. Validate that the input file path exists and is a single local document.
2. Resolve API key from CLI flag or environment.
3. Run the CLI to upload the file, execute OCR, and write markdown output.
4. Use the generated markdown file in follow-up agent steps.

## Run The CLI

Use `npm` from `scripts/`.

```bash
npm install
npm run build
node dist/cli.js <input-file> [--output <output-md>]
```

Examples:

```bash
node dist/cli.js ../sample_files/example.pdf
node dist/cli.js /any/path/to/example.pdf --output-dir ../out
node dist/cli.js ../sample_files/example.pdf --output ../out/example.md
node dist/cli.js ../sample_files/example.png --pages 0,1,2
```

## API Key Configuration

Provide Mistral API key in this precedence order:

1. `--api-key <key>`
2. `MISTRAL_API_KEY` in environment
3. `.env` file (auto-loaded from `--env-file` or nearest `.env` in current/parent directories)

Prefer `MISTRAL_API_KEY` in environment for CI/automation and `.env` for local development.

## Output Contract

Write one markdown file.

- Keep page order.
- Insert page separators between pages.
- Preserve OCR markdown as returned by the API.
- Default output path: `./ocr-output/<input-basename>.md` (relative to current working directory).
- Use `--output` for an exact file path or `--output-dir` for a custom output folder.

## Parameters

- `--model <name>`: OCR model (default `mistral-ocr-latest`)
- `--output-dir <path>`: output directory when `--output` is not specified
- `--pages <csv>`: zero-based page indexes
- `--keep-uploaded-file`: skip cleanup of uploaded file in Mistral storage
- `--include-image-base64`: request image payloads from OCR response

## References

Read `references/mistral-ocr-api.md` when changing OCR request fields or response handling.

---
> Source: [aikarjal/mistal-skill](https://github.com/aikarjal/mistal-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
