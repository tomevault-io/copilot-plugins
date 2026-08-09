## youtube-music-cli

> Project agent conventions for youtube-music-cli


# youtube-music-cli agent rules

- Read root `AGENTS.md` before non-trivial changes.
- Runtime is Bun; tests are `bun:test` under `tests/`.
- Prefer local playback helpers in `source/utils/local-track.ts` when touching play/download.
- Canonical agent skills live in `.agents/skills/`; after skill edits run `bun run sync:skills`.
- Implementation plans go in `docs/superpowers/plans/` (not `plan/` or `Plans/`).
- Do not add project-level `.mcp/` configs — configure MCP in the editor.
- Multi-step work: use planning-with-files / writing-plans skills.

---
> Source: [involvex/youtube-music-cli](https://github.com/involvex/youtube-music-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-09 -->
