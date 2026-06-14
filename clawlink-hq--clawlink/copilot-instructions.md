## clawlink

> <!-- BEGIN:nextjs-agent-rules -->

<!-- BEGIN:nextjs-agent-rules -->
# This is NOT the Next.js you know

This project runs **Next.js 15.5 (App Router) via OpenNext on Cloudflare Workers** — APIs, conventions, and file structure may differ from your training data. Before writing Next.js code, confirm the installed version in `node_modules/next/package.json` and verify against the official Next.js 15 docs (https://nextjs.org/docs). Heed deprecation notices.
<!-- END:nextjs-agent-rules -->

# Product Context

ClawLink is a SaaS for OpenClaw users. Prioritize simplicity, fewer steps, and language non-technical users can follow quickly. Hosted flows over manual setup. See `CLAUDE.md` for full product, deployment, and architecture details.

# Shared Context Vault (Obsidian)

The project includes an Obsidian vault at `notes/` for shared thinking space. All agents can read files from this directory. The user may reference notes by path (e.g., `notes/02-Projects/oauth-v2.md`).

Key files to know about:
- `notes/ClawLink Dashboard.md` — active work hub
- `notes/Project Map.md` — architecture overview, key concepts
- `notes/Decisions Log.md` — finalized decisions (tagged `#decision`)
- `notes/Open Questions.md` — open research questions (tagged `#question`)
- `notes/Multi-AI Workflow.md` — how the user works with multiple AI tools

If the user references a note, read it for context before answering.

# Deploy Commands

- Build: `npm run build:web`
- Deploy: `npm run deploy:web`

Schema or connection lifecycle changes require a migration + redeploy.

# Integration Data Model

- `user_integrations.id` is the stable connection id — users can have multiple rows per integration slug.
- `is_default` selects the implicit connection when `connectionId` is omitted from a tool call.
- `external_account_id` matches/reuses upstream accounts on reconnect.
- Row deletion is by connection id, not by slug. Do not reintroduce one-row-per-provider logic.
- Delete endpoint: `DELETE /api/connections/[id]`

# Integration MVP Testing

Validate in two layers before treating an integration as working:

1. **Connect flow** — OAuth login/consent/callback or manual credential submission ends with session `connected`.
2. **Tool execution** — reads/lists first, then writes. Use the hosted runtime, not a stub.

```bash
npm run test:openclaw-plugin-contract
npm run smoke:openclaw-plugin -- --preset <preset>
```

# Composio Integrations

All active integrations use the `composio` backend. See `CLAUDE.md` for architecture details on lazy schema hydration, BYO vs Managed OAuth, and the placeholder/format trap pattern.

Full scope audit log and per-provider findings: `openspec/specs/composio-scope-audit/spec.md`

Key rules:
- Default to BYO OAuth credentials for new integrations, not Composio Managed.
- Verify scope gaps by calling the tool directly through Composio before assuming a ClawLink bug.
- Check args for placeholder/format traps before assuming a missing-scope error.
- Use `fieldValidators` in `config/composio-tool-overrides.mjs` to enforce arg rules at the boundary.

# i18n and Translations

Full spec: `openspec/specs/i18n-translations/spec.md`

Supported locales: `en`, `es`, `pt`, `zh`. Marketing uses URL prefix (`/[locale]/...`); dashboard uses the `NEXT_LOCALE` cookie (no `[locale]` segment).

Key rules:
- Every new user-facing string must go through `useTranslations()` / `getTranslations()`. Do NOT hardcode English in components.
- Any PR that adds a new page, modal, email, or dashboard section MUST add the matching keys to **all** locale files in `src/messages/{en,es,pt,zh}.json` in the same PR. Missing keys render as the raw key path to users — there is no compile-time check.
- Adding a new locale requires four coordinated changes in one PR: `routing.locales`, both `LABELS` maps (marketing + dashboard toggle), a new `src/messages/<code>.json`, and a render-check. See the spec.
- Locale roadmap priority: `ja` → `de` → `fr` next. Defer `ar` until RTL layout work is committed.

# OpenClaw Plugin Releases

Full release workflow: `openspec/specs/openclaw-plugin-release/spec.md`

Key rules:
- Tag-based CI publishes to **both** npm (`@useclawlink/openclaw-plugin`) and ClawHub (`clawlink-plugin`).
- Tag format: `openclaw-plugin-v<version>`
- Keep these four in sync on every bump: `package.json`, `openclaw.plugin.json`, `public/skill.md`, `USER_AGENT` constant + test assertion.
- Do NOT run manual ClawHub publish after tagging — CI handles it.

---
> Source: [ClawLink-HQ/clawlink](https://github.com/ClawLink-HQ/clawlink) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-14 -->
