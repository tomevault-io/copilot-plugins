## premiere

> This repo is an Adobe Premiere MCP editor workspace. The active product is

# Agent Instructions

This repo is an Adobe Premiere MCP editor workspace. The active product is
`apps/premiere`.

## Read First

1. `README.md`
2. `.agents/memory/premiere-workflow.md`
3. `apps/premiere/skills/premiere-mcp-ops/SKILL.md`

## Active Paths

- `apps/premiere/adobe-mcp/` - Adobe MCP server, proxy, and UXP plugin workspace.
- `apps/premiere/scripts/legacy-extendscript/` - Quarantined legacy `.jsx`, not
  wired into anything. Use the MCP tools instead.
- `apps/premiere/skills/` - Repo-local workflow skills.

## Workflow Rules

- The live Premiere Pro timeline is the source of truth.
- Preflight first: `premiere_preflight()` checks proxy/plugin/project/sequence
  in one read-only call.
- Plan first: `remove_silence_segments(..., dry_run=True)` returns the exact cut
  plan without touching the timeline. Get user approval, then execute.
- Use `remove_silence_segments` for transcript-based removal ranges. It cuts with
  Premiere's Extract (ripple-delete), which normally closes the gap in the same
  A/V-synced op, confirms each Extract against the live layout, then verifies and
  returns a `verified` flag.
- Verify real sequence changes after every cut batch. Treat `verified: false` or
  `verified: null` as NOT confirmed — re-inspect with `verify_sequence_layout`.
- If Premiere/UXP returns no frame ticks and Extract leaves only tiny native
  gaps, run `close_gap_recovery(sequence_id)` — the automated bounded Close Gap
  recovery, one pass at a time, verified after every pass. Keep the cut only
  when it reports `clean: true`; otherwise undo to the previous clean baseline
  and stop.
- Stop on uncertain focus, wrong sequence, failed verification, or unsafe
  fallback. The canonical unsafe-tool list is in
  `apps/premiere/skills/premiere-mcp-ops/SKILL.md`.
- Do not create alternate sequences, rendered assemblies, or proxy edits unless
  the user explicitly asks for them.

## Checks

```bash
bun run premiere:check
bun run premiere:lint
bun run format:check
```

---
> Source: [shipshitshow/premiere](https://github.com/shipshitshow/premiere) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-17 -->
