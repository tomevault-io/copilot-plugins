## mythos

> Monsterly is an offline-first PWA for gym and CrossFit subscription control.

# AGENTS.md

## Project

Monsterly is an offline-first PWA for gym and CrossFit subscription control.

The app helps the operator quickly answer:

- Who is paid up?
- Who is about to expire?
- Who is already overdue?

Use [`monsterly.md`](monsterly.md) as the product source of truth. Do not
duplicate the brief in code or docs unless the task explicitly asks for it.

## Product Rules

- Keep the MVP simple and operational.
- Optimize for fast subscriber/payment-status control.
- Do not add payment processing, invoices, WhatsApp/SMS/email automation,
  complex analytics, or fancy branding unless the issue explicitly calls for it.
- Prefer clear mobile-friendly workflows over decorative UI.
- Status labels should stay aligned with the traffic-light model:
  `Al corriente`, `Por vencer`, and `Vencido`.

## Tech Stack

- Vite React SPA.
- TypeScript.
- pnpm.
- Tailwind CSS.
- shadcn/ui-style primitives in `src/components/ui`.
- React Router browser routing.
- Static host fallbacks live in `vercel.json` and `public/_redirects`.

Current routing is intentionally browser-routing based. The fuller migration to
React Router's `createBrowserRouter` is deferred until a route-data need arises.

## Code Style

- Use `@/*` imports for source aliases.
- Prefer existing shadcn/ui primitives before creating new component patterns.
- Use `cn()` from `src/lib/utils.ts` for conditional class composition.
- Prefer Tailwind utilities and design tokens from `src/styles.css`.
- Keep global CSS limited to Tailwind imports, theme tokens, resets, and tiny
  cross-component helpers.
- Build real app surfaces, not marketing landing pages.
- Keep UI restrained, accessible, and useful on mobile.
- Avoid complex dashboard behavior until the MVP needs it.

## Workflow

- Pull issue context from Linear before implementing issue work.
- Use the branch name from Linear unless the user overrides it.
- Keep commits atomic and conventional.
- For current review/fix work, use:

  ```sh
  git commit -m "fix(gpt5.5): [short description]" -m "[long description]"
  ```

- Do not push unless the user explicitly asks.
- Do not rewrite, reset, or revert user changes without explicit permission.
- If the worktree is dirty, inspect the changes and preserve unrelated user work.
- Prefer small, scoped changes over broad refactors.

## Verification

Run relevant checks before finishing:

```sh
pnpm build
pnpm lint
pnpm format:check
```

For docs-only changes, `git diff --check` is usually enough.

For route or layout work:

- Start the local dev server with `pnpm dev`.
- Smoke-check expected route URLs.
- Confirm direct route visits still work through the SPA fallback strategy.

Current shadcn variant helper exports may produce non-blocking Fast Refresh
warnings during `pnpm lint`. Treat errors as blockers; mention warnings in the
final summary when they remain.

## Do Not

- Do not add new libraries when existing stack patterns are enough.
- Do not switch routing strategy away from browser routing unless an issue
  explicitly calls for it.
- Do not introduce a public landing page unless explicitly requested.
- Do not store derived subscription status as permanent data unless the product
  plan changes.
- Do not hide verification failures. Report what failed and why.

---
> Source: [the-evillevi/mythos](https://github.com/the-evillevi/mythos) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-31 -->
