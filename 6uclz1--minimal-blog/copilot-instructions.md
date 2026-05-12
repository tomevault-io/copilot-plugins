## minimal-blog

> - Work test-first when changing behavior.

# Agent Instructions

## Development Style

- Work test-first when changing behavior.
- Keep runtime `dependencies` limited to `hono` unless the architecture is explicitly revised.
- Use handwritten CSS in `src/styles.css`; do not introduce Tailwind CSS.
- Write public repository documentation and commit messages in English.

## Architecture Rules

- Treat GitHub Issues as an external CMS adapter.
- Normalize CMS data into the `Post` domain model before it reaches routes or presentation components.
- Keep presentation components dependent only on domain types and shared helpers.
- Keep GitHub-specific code under `src/cms/github-issues`.
- Keep Markdown rendering and sanitization under `src/markdown`.
- Keep SEO output generation under `src/seo`.
- Keep build orchestration under `src/build`.

## Dependency Direction

Allowed high-level flow:

```text
GitHub Issues -> CMS adapter -> Post -> Markdown -> Hono SSG -> GitHub Pages
```

Forbidden dependencies:

- `presentation -> cms/github-issues`
- `content/domain -> markdown`
- `content/domain -> app`
- `content/domain -> GitHub API`

## Forbidden Changes

- Do not introduce Next.js.
- Do not introduce a database.
- Do not pass raw GitHub Issue objects into routes or presentation components.
- Do not bypass Markdown sanitization.
- Do not leave placeholder URLs.
- Do not introduce Tailwind CSS.
- Do not add runtime dependencies besides Hono without revising the architecture.

## Checks

Run these before finishing implementation work:

```sh
npm test
npm run typecheck
npm run lint
npm run build
```

---
> Source: [6uclz1/minimal-blog](https://github.com/6uclz1/minimal-blog) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-11 -->
