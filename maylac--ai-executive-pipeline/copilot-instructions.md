## ai-executive-pipeline

> ai-executive-pipeline is a Next.js app that reviews business ideas through a multi-CEO strategy pipeline.

# Agent Guide

## Purpose

ai-executive-pipeline is a Next.js app that reviews business ideas through a multi-CEO strategy pipeline.

## Structure

- `app/` contains the UI and API routes.
- `lib/` contains agent definitions and shared utilities.
- `tests/` contains route and library tests.
- `public/` contains static assets.

## Commands

- `npm run dev` starts the local app.
- `npm run build` verifies the production build.
- `npm test` runs the Node test suite when available in `package.json`.

## Guardrails

- Do not commit `.next/`, local env files, or generated cache artifacts.
- Keep prompts and CEO/persona behavior deterministic enough for tests.
- Treat external API keys and model settings as environment configuration.

---
> Source: [maylac/ai-executive-pipeline](https://github.com/maylac/ai-executive-pipeline) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
