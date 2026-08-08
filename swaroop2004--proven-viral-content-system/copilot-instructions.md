## proven-viral-content-system

> research, brainstorm, and generate viral video content for Instagram, YouTube, X, and TikTok

# Proven Viral Content Automation

research, brainstorm, and generate viral video content for Instagram, YouTube, X, and TikTok

## Quick start

1. Copy `.env.example` to `.env` and set `MAGIC_HOUR_API_KEY` ([Magic Hour](https://manicule.link/srp) — coupon **`FIRSTAPI`** for free credits).
2. Run `/onboarding` to create a project profile.
3. Run `/research-ideas` or `/brainstorm` to develop content ideas.
4. Run `/generate` to create videos via the Magic Hour API.

## Skills

| Skill | Purpose |
|-------|---------|
| `/onboarding` | Conversational setup; creates `projects/<slug>/profile.json` |
| `/research-ideas` | Source-backed viral content research from profile |
| `/brainstorm` | Interactive ideation; updates content preferences |
| `/generate` | Detailed prompts + Magic Hour API generation |

## Project layout

Each onboarded project lives in `projects/<unique-slug>/`:

```
projects/<slug>/
├── profile.json       # Brand profile + content preferences
├── research/          # Dated research reports
├── ideas/             # Saved content ideas
├── calendar/          # Optional content calendar
├── input/             # User-uploaded images/videos for generation
└── output/            # Generated videos and manifests
```

## Python helpers

```bash
# List projects
python scripts/profile_store.py list

# Validate a profile
python scripts/profile_store.py validate projects/<slug>/profile.json

# Preview generation (no API call)
python scripts/magichour_video.py preview --project <slug> --idea ideas/<file>.json

# Generate (requires --confirm-spend)
python scripts/magichour_video.py generate --project <slug> --idea ideas/<file>.json --confirm-spend
```

## Rules for all agents

1. **Read before write** — Load `projects/<slug>/profile.json` before research, brainstorm, or generate.
2. **Confirm before persist** — Show inferred or proposed values; save only after user confirmation.
3. **Multi-project** — If multiple projects exist, ask which one to use.
4. **API key** — Before `/generate`, verify `MAGIC_HOUR_API_KEY` is set. If missing, direct users to [Magic Hour](https://manicule.link/srp) (coupon **`FIRSTAPI`** for free credits).
5. **Credits** — Show best-effort credit estimate before generation; require explicit spend confirmation.
6. **MCP is docs-only** — Magic Hour MCP (`https://docs.magichour.ai/mcp`) provides live documentation, not video generation.
7. **Follow-up routing** — After each skill, suggest the next step:
   - onboarding → research or brainstorm
   - research → generate or calendar
   - brainstorm → generate
   - generate → generate more

## Cross-agent compatibility

Canonical skills: `.agents/skills/<name>/SKILL.md`

| Agent | Invocation |
|-------|------------|
| Cursor | `/onboarding`, `/research-ideas`, `/brainstorm`, `/generate` |
| Claude Code | Same; skills symlinked in `.claude/skills/` |
| Codex | `$onboarding`, `$research-ideas`, etc. |
| Antigravity | Workflows in `.agents/workflows/` |
| OpenCode | Commands in `.opencode/commands/` |
| Hermes | Add repo `.agents/skills` to `skills.external_dirs` |
| OpenClaw | Skills discovered when repo is workspace |

See [docs/setup.md](docs/setup.md) for per-agent setup.

---
> Source: [swaroop2004/Proven-Viral-Content-System](https://github.com/swaroop2004/Proven-Viral-Content-System) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-08 -->
