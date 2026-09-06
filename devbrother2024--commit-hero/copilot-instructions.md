## commit-hero

> Commit Hero는 한국어 대상 개발자 바이럴 웹서비스다. 사용자가 GitHub username을 입력하면 공개 GitHub 데이터와 OpenAI 밈 생성 결과를 조합해 RPG 게임 카드 형식의 공유용 개발자 프로필을 만든다.

# Commit Hero

## Project Context & Operations

### Overview
Commit Hero는 한국어 대상 개발자 바이럴 웹서비스다. 사용자가 GitHub username을 입력하면 공개 GitHub 데이터와 OpenAI 밈 생성 결과를 조합해 RPG 게임 카드 형식의 공유용 개발자 프로필을 만든다.

### Tech Stack
- Framework: Next.js App Router 16, React 19
- Language: TypeScript strict mode
- Styling: Tailwind CSS v4 via PostCSS
- Runtime/Package Manager: bun
- Integrations: GitHub public REST API, OpenAI SDK, Vercel production deployment
- UI Utilities: lucide-react, html-to-image
- Validation: ESLint, production build

### Operational Commands
- **Dev**: `bun run dev`
- **Build**: `bun run build`
- **Start**: `bun run start`
- **Test**: not configured; use `bun run lint` and `bun run build` as required validation
- **Lint**: `bun run lint`

## Golden Rules

### Immutable
- API keys, tokens, and secrets must never be committed.
- `.env.local` stays local and gitignored. Keep `.env.example` placeholder-only.
- Do not expose server secrets with `NEXT_PUBLIC_`.
- GitHub analysis must use public data only unless a new approved plan changes the product policy.
- OpenAI calls must stay server-side.
- Use `bun`; do not introduce npm, yarn, or pnpm lockfiles.

### Do's
- Keep the product language Korean-first for user-facing copy.
- Route browser requests through this app's API routes instead of calling GitHub or OpenAI directly from client components.
- Preserve graceful fallbacks for OpenAI failures and partial GitHub data failures.
- Treat GitHub rate limits as a product constraint; prefer caching, fewer upstream calls, and clear retry messaging.
- Keep response payloads typed through `src/lib/hero-types.ts`.
- Validate behavioral changes with `bun run lint` and `bun run build`.

### Don'ts
- Do not add auth, private repository access, OAuth, database storage, or billing without a new approved plan.
- Do not log or display API keys, tokens, raw provider errors containing sensitive data, or full environment values.
- Do not replace deterministic fallback behavior with hard failures when AI generation fails.
- Do not let UI components fetch third-party APIs directly.
- Do not make broad visual rewrites while changing API or analysis behavior.

## Standards & References

### Coding Conventions
- Use `@/*` imports for source files.
- Prefer small typed functions in `src/lib` and keep UI rendering in `src/components`.
- Use App Router conventions for pages and route handlers.
- Client components require `"use client"` only when they need browser APIs, state, or effects.
- Keep cards and controls compact with 8px-or-less radius unless the existing visual system changes.

### Git Strategy
- Branch: `main` deploys to Vercel production; use `feature/*`, `fix/*`, or `chore/*` for isolated work.
- Commit: imperative summary, scoped to the actual change.

### Maintenance Policy
규칙과 실제 코드 사이에 괴리가 발생하면 이 파일의 업데이트를 제안하라.

## Context Map

- **[App Router pages and API routes](src/app/AGENTS.md)** -- Next.js routing, metadata, and server route handler rules.
- **[Domain analysis and provider integrations](src/lib/AGENTS.md)** -- GitHub fetching, hero analysis, OpenAI generation, and shared response types.
- **[Reusable UI and sharing components](src/components/AGENTS.md)** -- Korean UI composition, card rendering, client fetching, copy, and image sharing behavior.

---
> Source: [devbrother2024/commit-hero](https://github.com/devbrother2024/commit-hero) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-06 -->
