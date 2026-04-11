## sift

> Sift is a script that generates a static web page to read RSS feeds, that could be run locally or deployed in GitHub Pages with GitHub Actions as cron job.

# Sift - Project Guidelines

## Project Overview

Sift is a script that generates a static web page to read RSS feeds, that could be run locally or deployed in GitHub Pages with GitHub Actions as cron job.

Features:

- Aggregation of content
- Filtering
- Auto categorization with NLP
- Content fetch with AI summary
- AI based ranking of content

## Architecture

The project has two distinct runtime modes:

- **Build-time generator** (`src/generate.ts`): Fetches RSS feeds, fetches full article content, runs AI/NLP processing, and outputs a self-contained static site to `dist/`.
- **Dev server** (`src/server.ts`): A lightweight Bun HTTP server for local development that serves the generated `dist/` folder.

### Generator Pipeline

```
config → fetch feeds → fetch content → categorize → summarize → rank → render HTML → write dist/
```

GitHub Actions runs the generator on a cron schedule and deploys `dist/` to GitHub Pages.

## Tech Stack

- Bun (runtime, package manager, and dev server)
- TypeScript (strict mode)
- Vercel AI SDK (`ai` package, provider-agnostic)

## Key Decisions

- **Vercel AI SDK**: Provider-agnostic AI via the `ai` package. Uses `@ai-sdk/openai-compatible` so any OpenAI-compatible API (OpenAI, Groq, Together, Ollama, etc.) works by setting `baseURL` in `sift.config.yaml`.
- **YAML config**: `sift.config.yaml` is the single source of truth for feed URLs, AI provider/model settings, and output options.
- **Content fetching**: Full article content is fetched and text-extracted before summarization, since RSS snippets are insufficient for quality summaries.
- **Static output**: The generator produces a self-contained `dist/` folder (HTML/CSS/JS). No runtime server is needed in production.

## Conventions

- **Formatting**: Prettier (enforced in CI)
- **Testing**: Vitest with V8 coverage (run with `bun run test:coverage`)
- **Type checking**: TypeScript strict mode, checked via `bun run typecheck`
- **Package manager**: Bun exclusively (no npm/yarn)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ricardovinicius)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/ricardovinicius)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
