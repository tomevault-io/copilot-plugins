## sprite-mobile

> For comprehensive documentation about sprite-mobile architecture, service management, development workflows, and sprite orchestration, see the sprite-mobile skill at `.claude/skills/sprite-mobile.md`.

# Claude Instructions for sprite-mobile

## Sprite-Mobile Skill

For comprehensive documentation about sprite-mobile architecture, service management, development workflows, and sprite orchestration, see the sprite-mobile skill at `.claude/skills/sprite-mobile.md`.

The skill covers:
- Architecture (tailnet-gate + sprite-mobile integration)
- Service management (restart procedures, logs, status)
- Development workflows (service worker cache versioning)
- Creating and managing other sprite-mobile sprites
- Configuration management and replication
- API endpoints and WebSocket protocol
- Common troubleshooting tasks

## Session Memories

Per-session memory summaries are stored as markdown files in `data/memories/`. Each file is named `{sessionId}.md` and contains a YAML frontmatter block (title, session ID, date) followed by a markdown summary of what was discussed, decided, and built in that session.

To review past work, read the files in `data/memories/`. You can also use the API:
- `GET /api/memories` — list all memories
- `GET /api/memories/:id` — read a specific memory
- `POST /api/memories/:id/generate` — auto-generate a summary from the conversation

## Git Commits

Do NOT add "Co-Authored-By" lines to commit messages. Just write normal commit messages without any co-author attribution.

---
> Source: [clouvet/sprite-mobile](https://github.com/clouvet/sprite-mobile) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-22 -->
