## general

> An open-source platform for managing and testing AI prompts with support for multiple LLM providers (OpenAI, Anthropic, Google Gemini).

# Genum Project

An open-source platform for managing and testing AI prompts with support for multiple LLM providers (OpenAI, Anthropic, Google Gemini).

## Project Structure

```
apps/core        		# Backend API server (Node.js/Express/TypeScript)
apps/web         		# Frontend React application (Vite/TypeScript)
packages/shared/ 		# Shared packages
docker-compose.*.yml  # Docker Compose files for different environments
```

## Tech Stack

**Backend (apps/core):** Node.js, TypeScript, Express.js, Prisma ORM, PostgreSQL, ClickHouse
**Frontend (apps/web):** React 18, TypeScript, Vite, Radix UI, Tailwind CSS, Monaco Editor, Zustand

## Package Manager

**Always use `pnpm` for all package management operations.** Never use npm or yarn. This is a pnpm workspace monorepo managed by Turbo.

---
> Source: [GenumAI/genum](https://github.com/GenumAI/genum) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-02 -->
