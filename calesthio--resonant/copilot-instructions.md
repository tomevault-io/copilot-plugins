## resonant

> When a user asks to compose, arrange, mix, analyze, or render music, use the `resonant` MCP tools. Do not edit `.resonant` JSON directly.

# Resonant repository guidance

When a user asks to compose, arrange, mix, analyze, or render music, use the `resonant` MCP tools. Do not edit `.resonant` JSON directly.

Follow this loop:

1. Call `get_capabilities` once per task.
2. For any lyric or song request, call `list_songwriting_languages`, match the requested language by ID, name, or alias, and follow that pack's coaching guide. Never silently fall back to English. Use the `write_song` prompt, analyze and revise the lyric, then save the draft before generation.
3. Create or inspect the target project and retain its revision.
4. Make one coherent revision-checked musical transaction at a time.
5. Re-inspect after a conflict; never force past a stale revision.
6. Validate and analyze the complete arrangement, respond to useful warnings, and render to a new WAV path.
7. Report the project path, WAV path, structural summary, analysis measurements, and any artistic choices that still need listening approval.

For software changes, preserve the delivery graph and run `npm run check`, `npm run build`, and the affected smoke tests. MCP changes additionally require `npm run mcp:smoke`.

---
> Source: [calesthio/Resonant](https://github.com/calesthio/Resonant) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-02 -->
