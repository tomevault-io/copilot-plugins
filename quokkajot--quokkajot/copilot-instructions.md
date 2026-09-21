## quokkajot

> Cross-cutting conventions for the whole monorepo. App-specific rules live in

# QuokkaJot — repo-wide rules

Cross-cutting conventions for the whole monorepo. App-specific rules live in
each app's own `CLAUDE.md` (`apps/*/CLAUDE.md`) — read this file first, then
the one for whatever app you're touching.

## Tooling — Bun only

- **Bun** is the package manager, script runner, and test runner. No npm, no
  pnpm, no Turborepo. `bun install`, `bun run --filter '<name>' <script>`.
- Prefer official CLIs over hand-written config (`wrangler`, `sv create`,
  `shadcn-svelte`, `better-auth generate`, `drizzle-kit`) over hand-rolled
  equivalents.
- Pin dependencies to current majors; don't downgrade to work around a bug
  without recording why (see the `typebox` override below).
- **Zod** (v4) is the validation library everywhere — remote functions, Hono
  routes, forms. Don't reach for a different schema library.

## Monorepo layout

Bun workspaces. Apps are deployables (Cloudflare Workers); packages are
libraries consumed as TypeScript source (no per-package build step, except
`api-client` and `ui` which are published).

```
apps/
  web        @quokkajot/web           SvelteKit — public board microsites (*.quokkajot.com + custom domains)
  dashboard  @quokkajot/dashboard      SvelteKit — authenticated admin app (app.quokkajot.com)
  landing    @quokkajot/landing        SvelteKit — marketing site (quokkajot.com apex)
  api        @quokkajot/api            Hono Worker — public REST API (api.quokkajot.com/v1)
  widget     @quokkajot/widget-host    Hono Worker — widget.js host + its public /v1 API (widget.quokkajot.com)
  email      @quokkajot/email-worker   Cloudflare Email Routing worker (inbound email → requests)
packages/
  config     @quokkajot/config         tsconfig + Tailwind presets, ESLint base
  db         @quokkajot/db             Drizzle schema + D1 client + migrations
  auth       @quokkajot/auth           Better Auth config (server + client), permissions.ts (ACL source of truth)
  core       @quokkajot/core           Domain services + validation (framework-free)
  shared     @quokkajot/shared         Framework-agnostic realtime client, schemas, server helpers (events, custom hostnames, kit)
  ui         @quokkajot/ui             Shared shadcn-svelte component set (consumed via dist by web/dashboard/landing)
  emails     @quokkajot/emails         Transactional email templates + sender
  api-client @quokkajot/api-client     Typed SDK for the public API (publishable to npm)
  widget     @quokkajot/widget         Embeddable widget frontend (Svelte custom element, Shadow DOM)
```

See [`docs/architecture/monorepo.md`](./docs/architecture/monorepo.md) for
dependency rules and rationale, and
[`docs/architecture/README.md`](./docs/architecture/README.md) for how the six
Workers fit together.

## Git

**STRICT:**

- Never create a git commit unless the user explicitly asks for one.
- Never add a Claude/AI co-author trailer to any commit.
- Never run destructive git commands (force-push, `reset --hard`, etc.)
  unless explicitly requested.

## UI

- shadcn-svelte components only — never raw `<select>`/`<input>`/`<button>`
  markup. See [`docs/architecture/design-system.md`](./docs/architecture/design-system.md)
  (STRICT, read before touching any `.svelte` file).
- Shared components live in `@quokkajot/ui`; run `bun run prepack` in
  `packages/ui` after editing it so consuming apps pick up the change from
  `dist`.
- Never inline a big UI block (dialog, panel, form) directly in a page file —
  always extract a component. Keep page files thin.
- Formsnap: `Form.Label` must live inside the `Form.Control` snippet — a bare
  sibling label crashes SSR.

## Comments

Default to no comments. Only add one when the WHY is non-obvious — a hidden
constraint, a subtle invariant, a workaround for a specific bug, behavior
that would surprise a reader. Never explain WHAT the code does when
well-named identifiers already say so.

```svelte
// Bad — this is obvious from the code, don't write it:
// Clear attachments whenever the composer reopens.
$effect(() => {
	if (newRequest.open) files = [];
});
```

If the logic is simple, leave it uncommented — no exceptions. Reserve
comments for the rare case where skipping one would actually mislead a
reader (e.g. why an effect is keyed off a transition instead of a
simpler-looking condition). When in doubt, delete the comment.

## Data layer — SvelteKit apps

`apps/web` and `apps/dashboard` use SvelteKit **remote functions**
(`query`/`command`/`form` from `$app/server`) to talk to our own backend.
Never hand-roll `fetch()` or a `+server.ts` route for first-party UI —
`+server.ts` is reserved for non-app consumers (webhooks, OAuth callbacks,
inbound email, third-party posts). `apps/landing` has no DB access and no
remote functions at all — it's static marketing content plus a session read
for its CTA; if that ever changes, the same remote-functions rule applies.
See each app's `CLAUDE.md` and
[`docs/architecture/frontend.md`](./docs/architecture/frontend.md).

## Positioning

QuokkaJot is described as "open-source, self-hosted alternative to hosted
feedback/roadmap/changelog tools" — generic category language only. Never
claim API compatibility with, or design/UI copying of, any specific named
competitor product, in docs, code comments, or UI copy.

## Local dev

- `.dev.vars` (git-ignored) per app for secrets; `wrangler.jsonc` per app is
  also git-ignored (real Cloudflare resource ids) — copy from that app's
  `wrangler.example.jsonc` template and fill in your own ids.
- Root `package.json` has a `typebox: 1.2.19` override — a newer typebox
  crashes superforms in dev. Don't remove it without re-verifying.
- See [`docs/guides/local-development.md`](./docs/guides/local-development.md)
  and [`docs/guides/deployment.md`](./docs/guides/deployment.md).

---
> Source: [quokkajot/quokkajot](https://github.com/quokkajot/quokkajot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-21 -->
