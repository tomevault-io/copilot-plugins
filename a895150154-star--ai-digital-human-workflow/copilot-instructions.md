## ai-digital-human-workflow

> - `video-agent/` is the primary AI video production project. New video work uses Remotion as the master timeline. The HyperFrames project directory has been removed; do not recreate it.

# Codex workspace guidance

## Workspace layout

- `video-agent/` is the primary AI video production project. New video work uses Remotion as the master timeline. The HyperFrames project directory has been removed; do not recreate it.
- `.agents/skills/` contains repo-scoped Codex skills used by this workspace.
- For any task that reads or edits `video-agent/`, read and follow `video-agent/AGENTS.md` first; its rules are more specific.

## Codex entry points

- Use `AGENTS.md` for durable instructions that Codex should load automatically.
- Use `README.md` for human-facing project documentation.
- Use `.agents/skills/<skill-name>/SKILL.md` for reusable workflows.
- Use `.codex/config.toml` only when an actual project-scoped Codex setting is needed. Do not create `CLAUDE.md` or `CODEX.md` as instruction files.

## Recent-topic research

- Use the repo skill `$last30days` when the user asks for recent trends, current audience discussion, topic discovery, or candidate topics for `video-agent/`.
- For this workspace, invoke it with the free-first process profile defined in `video-agent/config/project.json`: `SETUP_COMPLETE=true`, `FROM_BROWSER=off`, and an absolute `LAST30DAYS_MEMORY_DIR` resolving to `video-agent/logs/topic-research`. Do not read browser cookies or enable paid providers unless the user explicitly changes that policy.
- Before fresh research, run `npm run check:research-network` from `video-agent/`. If it reports `NETWORK_ACCESS_REQUIRED`, do not run the engine in the same restricted sandbox and do not present an empty trend result; request approved external-network execution and rerun the identical `last30days` command once. Keep that same network-authorized execution context for every discovery leg that fetches evidence, including the post-judgment enrichment leg.
- Treat `URLError` DNS failures or runs where every discovery seed source is degraded as execution failures, not as evidence that no topic exists. Only accept `Nothing solid this window` as a content result when the required discovery feeds passed the network check.
- Treat Reddit, Hacker News, GitHub, and other community engagement as discovery evidence, not as proof that a claim is true. Before a claim enters a script, verify it against an official or first-party source and record that verification in the run's `research.md`.
- If the user asks only for candidate topics, stop after presenting the requested candidates and wait for a selection. Do not continue into scriptwriting or video generation.

## Validation

After changing `video-agent/`, run from that directory:

```bash
npm run check:init
```

After changing a project skill, run the built-in skill validator against that skill folder. Preserve upstream third-party behavior unless the change is required for Codex compatibility or explicitly requested.

---
> Source: [a895150154-star/ai-digital-human-workflow](https://github.com/a895150154-star/ai-digital-human-workflow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-27 -->
