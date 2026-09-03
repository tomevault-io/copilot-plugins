## hermit

> These instructions apply to this repo.

# AGENTS

These instructions apply to this repo.

- Use Carbon components v2 (`TextDisplay`, `Container`, `Section`) instead of `content` or embeds.
- Register new commands/listeners in `src/index.ts`.
- Keep automod response templates in `src/config/automod-messages.json`.
- Commands must declare integration types/interaction contexts explicitly: snippet commands should allow user installs, moderation commands must stay guild-only.
- Use `bun`/`bunx` for package and script commands; do not use pnpm.
- Keep form definitions config-driven in root `forms.config.ts`; no backoffice UI.
- `forms.config.ts` is the production source of truth for form IDs, copy, review channels, and accept/deny actions; keep actions wired before calling forms prod-ready.
- Put shared Forms types in `src/forms/types.ts`; do not define config types in `forms.config.ts`.
- Prefer React SSR for web pages; avoid hand-built HTML strings for substantial views.
- For Discord review messages, use Carbon components/buttons instead of raw component payload objects.
- Cloudflare Builds deploys `main`; do not manually run prod deploys unless explicitly asked. Push to `main` after verification and let Cloudflare deploy.

---
> Source: [openclaw/hermit](https://github.com/openclaw/hermit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-03 -->
