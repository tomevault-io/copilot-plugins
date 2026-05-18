## cihal

> AI-powered English speaking practice platform for OPIc exam preparation.

# CIHAL - OPIc IH/AL Learning Service

AI-powered English speaking practice platform for OPIc exam preparation.

## Tech Stack

| Layer | Technology |
|-------|------------|
| Web | Next.js, React, TypeScript, Tailwind CSS |
| Mobile | React Native, Expo |
| API | Spring Boot, Java 21, PostgreSQL |
| AI | FastAPI, Python, LangChain, OpenAI |

## Directory Structure

- `apps/web/` - Next.js web application
- `apps/mobile/` - React Native Expo app
- `backend/api/` - Spring Boot REST API
- `backend/ai/` - FastAPI AI/ML server
- `packages/ui/` - Shared UI components
- `docs/` - Project documentation

## Workflow

### Build & Test
- **Spring Boot (backend/api)**: Use Docker for build/test (local Java is 17, project requires 21)
  ```bash
  cd backend/api && docker build -t cihal-api-test .
  ```
- **Python (backend/ai)**: Use `python3` command (not `python`)

### Git Commits
- Use git-commit-helper agent via Task tool for commit analysis
- Never perform commit analysis directly in main agent

---
> Source: [sequaa/cihal](https://github.com/sequaa/cihal) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
