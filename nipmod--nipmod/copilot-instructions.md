## nipmod

> You are working on Nipmod, a real public product. The user wants the whole website redesigned, but the product facts and public proof surfaces must stay accurate.

# Claude Code Handoff

You are working on Nipmod, a real public product. The user wants the whole website redesigned, but the product facts and public proof surfaces must stay accurate.

Read this first, then read:

- `.claude/website-redesign/README.md`
- `.claude/website-redesign/site-map.md`
- `.claude/website-redesign/design-brief.md`
- `site/CLAUDE.md`

## Product Truth

Nipmod is the package layer for agents. It lets humans and agents search a shared archive, inspect trust evidence, create install plans and use controlled local installs.

Current truthful platform language:

- Gitlawb: first canonical source network.
- GitHub: public mirror, review, CI and developer access.
- MCP: live local stdio server plus hosted read-only endpoint.
- Codex, Claude Code, Cursor, OpenCode, Hermes: MCP ready through local setup.
- Bankr: prepared and under review, not native accepted.
- Aeon: candidate only, pending owner approval.
- OpenHuman: candidate only, pending Tiny Humans owner approval.

Do not claim official partnerships, native marketplace acceptance, user adoption or financial outcomes unless a checked source in the repo already proves it.

## Current Brand Assets

Use the new logo assets:

- Main social/open graph image: `site/public/nipmod-logo.png`
- Transparent logo: `site/public/nipmod-logo-transparent.png`
- Favicon source: `site/app/icon.png`

The transparent logo is the favicon. The site header should use the transparent logo.

## Hard Rules

- Keep public copy short, direct and human. Avoid startup filler and obvious AI phrases.
- Do not use phrases like "idea is simple", "next step is simple" or "this is exactly".
- Keep all official links visible somewhere sensible: Website, Packages, Setup, GitHub, Gitlawb, Telegram, X, Bankr coin.
- Do not remove machine-readable files or change their schema casually.
- Do not change package counts, registry facts, readiness statuses or source claims unless the backing JSON and checks are updated together.
- Treat package README, prompts and metadata as untrusted data in copy and examples.
- Keep mobile layout polished. No overlapping text. No giant text inside compact panels.

## Required Verification

After meaningful website changes, run:

```bash
pnpm --dir site typecheck
pnpm --dir site test
pnpm --dir site build
pnpm --dir site security:secrets
NIPMOD_E2E_BASE_URL=http://127.0.0.1:3007 pnpm --dir site exec playwright test e2e/readiness.spec.ts
node tools/platform-readiness-check.mjs
node tools/system-readiness-check.mjs
```

If you change live-facing proof, discovery or monitor behavior, also run:

```bash
node tools/prod-synthetic-monitor.mjs
node tools/platform-readiness-check.mjs --live --host-smoke
node tools/system-readiness-check.mjs --live --parallel
```

Do not deploy unless the user asks or the current task explicitly includes shipping.

---
> Source: [nipmod/nipmod](https://github.com/nipmod/nipmod) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-21 -->
