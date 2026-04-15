## play-prediction

> All instructions for AI agents are documented in AGENTS.md files throughout this monorepo.

# Gemini Instructions

All instructions for AI agents are documented in AGENTS.md files throughout this monorepo.

## 📁 File Locations

| File | Content |
|------|---------|
| [`AGENTS.md`](./AGENTS.md) | Root instructions, critical rules |
| [`backend/AGENTS.md`](./backend/AGENTS.md) | Backend patterns, DI, errors |
| [`frontend/AGENTS.md`](./frontend/AGENTS.md) | Frontend patterns, routing |
| [`docs/AGENTS.md`](./docs/AGENTS.md) | Documentation navigation |

## 🚨 Critical Reminders

1. **Use `requireEnv('VAR_NAME')`** instead of `process.env.VAR_NAME`
2. **Use drizzle-kit** for migrations, never Supabase CLI
3. **Never use Supabase browser client** - all calls go through backend

Start with the [root AGENTS.md](./AGENTS.md) for the full overview.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/blackpr)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/blackpr)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
