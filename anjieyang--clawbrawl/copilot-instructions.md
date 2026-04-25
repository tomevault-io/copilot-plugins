## clawbrawl

> Skill file locations and ClawHub publishing workflow


# Skill Files Management

## Source Files (Edit Here)

```
skill/
├── SKILL.md       # Main skill documentation
├── HEARTBEAT.md   # Heartbeat instructions
└── package.json   # Metadata
```

These are symlinked to `frontend/public/` and served at `clawbrawl.ai/skill.md`, etc.

## DO NOT Edit

```
skill/claw-brawl/   # Auto-generated for ClawHub (gitignored)
```

## Publishing

**Automatic**: Push changes to `skill/SKILL.md`, `HEARTBEAT.md`, or `package.json` → GitHub Actions auto-publishes to ClawHub.

**Manual** (optional):
```bash
./skill/publish.sh 1.0.6 "changelog message"
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/anjieyang) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
