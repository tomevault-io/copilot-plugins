## theirspace

> This document provides **engineering and product-quality standards** for building polished software across web, CLI, desktop, and mobile. The primary target is **web apps deployed to Vercel** using modern tooling (Bun, TypeScript, Next.js), but the standards apply broadly.

# Agent Guidelines (General-Purpose, Web-First)

This document provides **engineering and product-quality standards** for building polished software across web, CLI, desktop, and mobile. The primary target is **web apps deployed to Vercel** using modern tooling (Bun, TypeScript, Next.js), but the standards apply broadly.

## North Star

Build **production-quality** software from day one:
- **Design + engineering excellence** > rapid MVP shortcuts
- **Shipping confidence** via testing, CI, and observability
- **Performance and accessibility** as core product features

## Preferred technology choices

- **Runtime**: Bun (preferred), Node 20+ as fallback
- **Package manager**: Bun or pnpm (avoid npm)
- **Frontend**: Next.js App Router, React Server Components, Vite for non-Next apps
- **Styling**: Tailwind or Vanilla Extract; design system first
- **Backend**: **Python** (FastAPI / Litestar) as preferred
- **Database**: Postgres + Prisma/Drizzle
- **Deploy**: Vercel for web; Docker + Fly.io/Render for services

## General guidelines

### Web
- Prefer **server components** and **edge runtime** where appropriate.
- Use **typed contracts** (tRPC, OpenAPI, Zod schemas).
- Implement **sensible caching** (stale-while-revalidate).

### CLI
- Use **Python (Typer)** or **TS (oclif)** for clean UX.
- Ensure **sane defaults**, `--help`, and error messages.

### Desktop / Mobile
- macOS: **SwiftUI** or **Tauri** (Rust + web UI).
- iOS: SwiftUI with async/await.

## Product polish checklist

- Micro-interactions and loading states are intentional.
- Empty states are designed and useful.
- Errors are informative and actionable.
- Accessibility is validated (keyboard + screen reader).
- Performance targets documented and tested.

## Testing expectations

- Unit tests for pure logic.
- Integration tests for APIs and DB.
- Playwright for critical user flows.
- Run tests in CI before merge.

## Deployment expectations (Vercel)

- Use Vercel env vars and secrets.
- Set runtime region(s) intentionally.
- Monitor logs, errors, and performance.

## Documentation expectations

Every project should include:
- `README.md` (product overview + stack)
- `ARCHITECTURE.md` (high-level system design)
- `CONTRIBUTING.md` (standards + workflow)
- `PROJECT_CHECKLIST.md` (launch readiness)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/tfpickard)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/tfpickard)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
