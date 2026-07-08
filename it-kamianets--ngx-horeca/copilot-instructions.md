## ngx-horeca

> This repository is an Angular 22 HoReCa marketing site that consumes reusable HoReCa contracts,

# Agent Guide

This repository is an Angular 22 HoReCa marketing site that consumes reusable HoReCa contracts,
services, and guidance from the npm package `@wawjs/ngx-horeca`.

Treat this project as a prerender-first business website. The app owns its pages, routes, layouts,
business content, company profile, static assets, and bootstrap wiring. Reusable HoReCa feature APIs
must be imported from `@wawjs/ngx-horeca`.

## Quick Reference

- Stack: Angular 22, TypeScript 6, Angular SSR/prerender, Tailwind CSS, SCSS
- Package manager: `npm`
- Main goal: fast, clean, SEO-friendly HoReCa landing pages
- Primary output: prerendered static site from `dist/app/browser`
- Local app source: `src/`
- App-owned data bridge and company profile: `src/app/feature/bootstrap/` and
  `src/app/feature/company/`
- Reusable package: `@wawjs/ngx-horeca`
- Package guidance: `node_modules/@wawjs/ngx-horeca/ai/`

## Package Guidance

After reading this file, load the package guidance from:

```text
node_modules/@wawjs/ngx-horeca/ai/
```

The package is scoped. Use `node_modules/@wawjs/ngx-horeca`, not
`node_modules/wawjs/ngx-horeca`.

Start with `node_modules/@wawjs/ngx-horeca/ai/onboarding.md`, then open only the guides needed for
the task:

- `node_modules/@wawjs/ngx-horeca/ai/architecture.md`
- `node_modules/@wawjs/ngx-horeca/ai/code-style.md`
- `node_modules/@wawjs/ngx-horeca/ai/content-pages.md`
- `node_modules/@wawjs/ngx-horeca/ai/seo.md`
- `node_modules/@wawjs/ngx-horeca/ai/media.md`
- `node_modules/@wawjs/ngx-horeca/ai/tooling.md`
- `node_modules/@wawjs/ngx-horeca/ai/task-execution.md`
- `node_modules/@wawjs/ngx-horeca/ai/decisions/index.md`

Read the decisions index when a task changes a long-lived convention, resolves an ambiguity likely
to recur, or affects package-facing behavior.

## Companion `@wawjs/*` Package Guidance

Installed `@wawjs/*` packages, except the local HoReCa package, may provide a package-level `AI.md`
file at:

```text
node_modules/@wawjs/<package>/AI.md
```

Before adding or changing code that uses one of these companion packages, read that package's
`AI.md` first. This is especially important when there is no local sample code, or the nearest
sample does not cover the required behavior.

Current companion package guides include:

- `node_modules/@wawjs/ngx-core/AI.md`
- `node_modules/@wawjs/ngx-http/AI.md`
- `node_modules/@wawjs/ngx-translate/AI.md`
- `node_modules/@wawjs/ngx-ui/AI.md`

For example, before introducing `HttpService` usage without a matching local pattern, read
`node_modules/@wawjs/ngx-http/AI.md`.

## Universal Rules

- Treat this repo as a marketing website first, not as a complex application shell.
- Prefer simple, static, content-first pages over heavy abstractions.
- Preserve prerender compatibility by default.
- Keep changes small, clear, and easy to review.
- Prefer Tailwind for layout, spacing, typography, sizing, responsive behavior, and utility styling.
- Use local page content/config over new services unless reuse is real and repeated.
- Do not introduce CMS, API fetching, dashboards, or heavy state management unless explicitly
  requested.
- Do not edit files inside `node_modules/@wawjs/ngx-horeca`; update the package source and publish a
  new package version instead.

## Default Technical Stance

Use these as defaults unless the local code or the task gives a concrete reason to do otherwise:

- Angular 22 modern patterns only.
- Standalone components are the default. Do not add NgModules for new work.
- Do not add `standalone: true`; standalone is already the default in Angular 20+.
- Use `@Service()` for root-provided services. Keep `@Injectable()` for cases that need deeper DI
  configuration.
- Use `changeDetection: ChangeDetectionStrategy.OnPush` on new or touched components.
- Use signals for local UI state and derived state.
- Prefer Angular Signal Forms for new form work.
- Prefer `httpResource()` or `@wawjs/ngx-http` `ngxHttpResource()` for new signal-based HTTP reads;
  guard resource `value()` reads with `hasValue()`.
- Prefer `resource()` or `@wawjs/ngx-core` `ngxResource()` for new non-HTTP async signal reads.
- Prefer native control flow (`@if`, `@for`, `@switch`) in templates.
- Use Angular 22 template spread and short arrow functions only for simple local UI glue.
- Use Angular bindings instead of manual DOM work.
- Use `NgOptimizedImage` for static images when feasible.
- Keep browser-only code guarded so prerender remains safe.

## Asset Paths

- Do not assume Angular assets are served from `/assets`.
- Treat `src/assets` as a source folder, not a runtime URL.
- Check `angular.json` before answering questions or editing templates that reference static files.
- In this repo, files from `src/assets` are published from the site root, so `src/assets/logo.webp`
  is referenced as `/logo.webp` or `logo.webp`, not `assets/logo.webp`.

## Import Rules

- Import reusable HoReCa APIs from `@wawjs/ngx-horeca`.
- Do not import package internals from app code.
- Keep pages, layouts, route definitions, prerender route lists, local data, company defaults,
  bootstrap fallback imports, and static assets app-owned under `src/`.

## Loading Order

1. `AGENTS.md`
2. `node_modules/@wawjs/ngx-horeca/ai/onboarding.md`
3. One or two relevant package `ai` guides for the task
4. Relevant companion `node_modules/@wawjs/<package>/AI.md` files before using those package APIs
5. `node_modules/@wawjs/ngx-horeca/ai/decisions/index.md` only if the task may affect durable policy

---
> Source: [IT-Kamianets/ngx-horeca](https://github.com/IT-Kamianets/ngx-horeca) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
