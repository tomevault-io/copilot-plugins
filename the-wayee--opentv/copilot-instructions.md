## opentv

> **[CLAUDE.md](CLAUDE.md) is the source of truth** for what this project is, its

# Repository Guidelines

**[CLAUDE.md](CLAUDE.md) is the source of truth** for what this project is, its
architecture, the full command list, the directory layout, and the hard-won gotchas.
Read it first. This file only adds the conventions that aren't covered there:
code style, testing habits, and commit/PR expectations.

## Orientation (30 seconds)

**OpenTV** — planning + producing serialized short-form video, in **two deliberately
separate forms**:
- **创意画布** (`app/canvas/[projectId]`) — one infinite React Flow canvas per project,
  human-in-the-loop, quality first. Nodes (media / script / text / 视频拆解) + a side Agent.
- **生产工作流** (`app/workflow/[projectId]`) — unattended LangGraph pipeline, one project =
  一档节目 (multi-episode), efficiency first. This is the current main front.

`(app)/` is the signed-in shell with a sidebar; both workspaces live outside it (full-screen).
Everything is org-scoped; every project route goes through `lib/projects/access.ts`.

**Work on the `dev` branch.** Pushing `main` deploys to production (CI gates on checks, then
rsyncs + rebuilds on the server), so only merge to main when the user says so.

Commands: `npm run dev` · `npm run lint` · `npm run typecheck` · `npm test` ·
`npm run build`. On a fresh clone run `npx prisma generate` first, or typecheck reports
phantom errors. Use **npm** — `package-lock.json` is the only lockfile.

## Coding Style & Naming

- TypeScript, React function components, two-space indent, **no semicolons**,
  **double quotes** — match the file you're editing rather than reformatting it.
- `@/` path imports. kebab-case filenames (`password-block.tsx`), PascalCase components,
  camelCase helpers.
- Reuse the Tailwind design tokens from `app/globals.css` (`bg-card`, `text-ink`,
  `border-border`, `text-vermilion`, …) — never hardcode colors. Icons from `lucide-react`.
- Keep server-only code out of client components; validate untrusted input at the API
  boundary (zod).
- **Comments explain *why*, in Chinese, and they earn their length.** The house style is a
  short block above the tricky thing recording the reasoning and the failure it prevents
  (see `lib/uid.ts`, `lib/invite.ts`, `.github/workflows/deploy.yml`). Don't strip these,
  and don't replace them with restatements of the code.

## Testing

Vitest + Testing Library, colocated as `*.test.ts` / `*.test.tsx`. Target validation logic,
security boundaries, and user interactions; mock only external I/O. No coverage threshold.

- One file: `npm test -- lib/invite` · one case: `npm test -- -t "邀请码"`.
- `*.integration.test.ts` files hit real APIs and **cost real money**. They self-skip unless
  their flag is set (`RUN_EVOLINK_IT`, `RUN_AGENT_IT`, `RUN_PRODUCE`, `RUN_GEN_VERIFY`,
  `RUN_SEED_SKILLS`), so plain `npm test` is always safe. Keep new ones gated the same way.
- Before a PR: `npm run lint`, `npm run typecheck`, `npm test`. CI runs exactly
  `prisma generate` → `prisma validate` → `typecheck` → `lint` → `test`. Don't run
  `npm run build` while a `next dev` server is up — they share `.next` and it breaks dev.

## Commits & PRs

Scoped Conventional Commits with a **Chinese** subject that says what changed and, when
there's a surprise, why — e.g. `fix(canvas): 线上点「添加节点」没反应——crypto.randomUUID 仅安全上下文可用`.
Common scopes: `canvas` `deploy` `ci` `docker` `render` `workspace` `feishu-env`.
Keep each commit focused. PRs should explain behavior and risk, list the validation
commands you ran, call out migrations or config changes, and include screenshots for
visible UI changes.

## Security & Configuration

- Copy `.env.example` → `.env`; never commit `.env` or credentials. Production config is
  generated from a 飞书 bitable (`npm run env:pull` / `env:push`).
- **AI provider keys are never in `.env`** — they're per-organization, AES-256-GCM encrypted
  in `ProviderCredential`, entered at 「设置 → AI 服务」. Any route that calls AI must be
  wrapped with `withProjectAi` (`lib/credentials/route.ts`).
- Registration is **invite-only** and fails closed in production (`lib/invite.ts`; the one
  choke point is `databaseHooks.user.create.before` in `lib/auth.ts`).
- Avatars and **generated media are currently uploaded public-read** — a deliberate,
  temporary deviation from the private + short-lived-signed-URL norm, to revisit when the
  asset library hardens. Don't assume assets are private; don't widen the exception either.
- Apply Prisma migrations before deploying. Any `schema.prisma` change needs a migration.

---
> Source: [the-wayee/OpenTV](https://github.com/the-wayee/OpenTV) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-28 -->
