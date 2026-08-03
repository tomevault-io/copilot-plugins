## trackfit

> TrackFit is a fitness tracking app. This file defines project standards for Claude Code sessions.

# TrackFit — Claude Code Configuration

TrackFit is a fitness tracking app. This file defines project standards for Claude Code sessions.

## UX UI Pro Max Skill

This project has the full UX UI Pro Max Skill installed. Available commands:

| Command | Description |
|---------|-------------|
| `/ux-review` | Run a full UX/UI audit on recent changes |
| `/design` | Create or update UI designs in Canva |
| `/design-sync` | Sync Canva designs with the codebase |
| `/ui-agent` | Build new UI components with best practices |
| `/feedback` | Generate a comprehensive UX feedback report |

## MCP Integrations

- **Canva** (`mcp__233f22b9-*`): Design creation, brand assets, export
- **Google Drive** (`mcp__19f0dad7-*`): Asset storage, design docs, reports
- **GitHub** (`mcp__github__*`): PRs, issues, code review

## Design System

### Colors
- Primary: `#2563EB` (blue) or `#16A34A` (green)
- Accent/CTA: `#EA580C` (orange)
- Backgrounds: `#F9FAFB` (light), `#111827` (dark)
- Success: `#22C55E`, Warning: `#F59E0B`, Error: `#EF4444`

### UX Principles
1. **Speed of logging** — Every workout/meal entry should take ≤10 seconds
2. **Visible progress** — Users must see their progress without navigating
3. **Positive reinforcement** — Celebrate every milestone, no matter how small
4. **Mobile-first** — 80%+ of fitness tracking happens on mobile
5. **Accessible** — WCAG AA minimum for all UI

## Automatic Feedback

The session-end hook will remind you to run `/feedback` after any UI changes.
The post-edit hook will flag UI file changes for review.

## Project Structure

```
TrackFit/
├── .claude/
│   ├── settings.json     # Permissions & hooks
│   └── commands/         # UX UI Pro Max Skills
│       ├── ux-review.md
│       ├── design.md
│       ├── design-sync.md
│       ├── ui-agent.md
│       └── feedback.md
├── CLAUDE.md             # This file
└── src/                  # App source code
```

## Git Workflow

- Branch: `claude/ux-ui-pro-max-skill-hqjHY`
- Always run `/ux-review` before committing UI changes
- Always run `/feedback` before opening a PR

---
> Source: [Relyt0801/TrackFit](https://github.com/Relyt0801/TrackFit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-26 -->
