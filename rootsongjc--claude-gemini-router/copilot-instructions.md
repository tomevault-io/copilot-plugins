## claude-gemini-router

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Common Commands

- **Development Server:** `npm run dev` (starts a local development server using Wrangler)
- **Deployment:** `npm run deploy` (deploys the worker to Cloudflare)
- **Unit Tests:** `npm run test` (runs unit tests, ignoring integration tests)
- **Integration Tests:** `npm run test:integration` (runs integration tests)
- **All Tests:** `npm run test:all` (runs all tests)
- **Watch Tests:** `npm run test:watch` (runs tests in watch mode)

## Code Architecture

This project is a Cloudflare Worker designed to act as a proxy between the Anthropic Claude API and the Google Gemini API.

- **`src/index.ts`**: The main entry point for the Cloudflare Worker, handling incoming requests and routing them.
- **`src/formatRequestGemini.ts`**: Contains logic for transforming incoming Anthropic API requests into the Google Gemini API format.
- **`src/formatResponseGemini.ts`**: Contains logic for transforming responses from the Google Gemini API back into the Anthropic API format.
- **`src/streamResponseGemini.ts`**: Handles the specifics of streaming responses between the two API formats.
- **`src/logger.ts`**: Provides logging utilities for the worker.
- **`src/types.ts`**: Defines shared TypeScript types and interfaces used across the project.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/rootsongjc) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
