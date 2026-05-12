## niteni

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Niteni is an AI-powered code review tool for GitLab CI pipelines, powered by Google's Gemini API. The name comes from Javanese, meaning "to observe carefully." It posts inline diff comments with suggestions directly on GitLab merge requests.

**Zero external runtime dependencies** — only uses Node.js built-in modules (`https`, `child_process`, `fs`, `path`, `os`, `url`) plus TypeScript for compilation.

## Commands

```bash
npm run build              # Compile TypeScript (tsc)
npm run dev                # Watch mode compilation
npm run lint               # ESLint on src/
npm run test               # Node built-in test runner (dist/**/*.test.js) — build first
npm run review:simulate    # Run simulation mode with mock data (needs ts-node)
npm run simulate           # Run simulation from compiled output
```

Tests use Node's built-in `node:test` and `node:assert` modules. Run `npm run build && npm test` to compile and execute tests from `dist/**/*.test.js`.

## Architecture

```
src/
├── cli.ts          # CLI entry point — three modes: mr, diff, simulate
├── index.ts        # Main orchestration: runMergeRequestReview() & runDiffReview()
├── reviewer.ts     # Gemini review logic with structured JSON output
├── gitlab-api.ts   # GitLab REST API client (MR data, inline comments, cleanup)
├── config.ts       # Environment variable parsing & validation
├── simulate.ts     # Self-contained demo with ANSI-colored mock output
└── types/
    ├── index.ts    # Barrel export
    ├── config.ts   # AppConfig, GitLabConfig, GeminiConfig, ReviewConfig
    ├── gitlab.ts   # MergeRequest, DiffPosition, MergeRequestNote
    └── reviewer.ts # Severity, Finding, StructuredReviewResponse, ReviewResult, FilterOptions
```

### Gemini Structured Output (reviewer.ts)

The `Reviewer` class uses the Gemini REST API with structured output (`responseMimeType: "application/json"` + `responseSchema`) to return typed `StructuredReviewResponse` objects directly — no regex parsing needed. The response schema enforces `summary` (string) and `findings` (array of `Finding` objects with severity, file, line, description, optional suggestion/rationale).

### MR Review Flow (index.ts)

`runMergeRequestReview()`: Fetch MR metadata → get diffs → filter by include/exclude patterns → call Gemini with structured output → receive typed findings directly → clean up old review comments → post inline diff comments with severity emojis and suggestion blocks.

### GitLab API Client (gitlab-api.ts)

Supports three auth modes via `GITLAB_TOKEN`: `PRIVATE-TOKEN`, `JOB-TOKEN` (CI default via `$CI_JOB_TOKEN`), and `Bearer` (OAuth). Uses Node.js `https` module directly.

### Review Output Format

Findings are returned as typed JSON objects via Gemini's structured output. Each `Finding` has: severity (`CRITICAL`|`HIGH`|`MEDIUM`|`LOW`), file path, line number, description, optional suggestion, and optional rationale.

## Key Environment Variables

| Variable | Default | Purpose |
|----------|---------|---------|
| `GEMINI_API_KEY` | Required | Google Gemini API key |
| `GITLAB_TOKEN` | `$CI_JOB_TOKEN` | GitLab auth token |
| `GEMINI_MODEL` | `gemini-3-pro-preview` | Model for REST API |
| `REVIEW_MAX_FILES` | `50` | Max files to review |
| `REVIEW_MAX_DIFF_SIZE` | `100000` | Max diff size in chars |
| `REVIEW_INCLUDE_PATTERNS` | — | Comma-separated glob patterns to include |
| `REVIEW_EXCLUDE_PATTERNS` | `package-lock.json,...` | Comma-separated glob patterns to exclude |
| `REVIEW_FAIL_ON_CRITICAL` | `false` | Exit 1 on CRITICAL findings |

## TypeScript Configuration

- Target: ES2022, Module: CommonJS
- Strict mode enabled
- Source maps and declaration files generated
- Output directory: `dist/`

---
> Source: [denyherianto/niteni](https://github.com/denyherianto/niteni) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-03 -->
