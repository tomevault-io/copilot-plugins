## tool-db

> Keep Markdown harness files synchronized with behavior changes


# Markdown Harness Sync

- Treat instruction-driving Markdown files as part of the implementation, not optional docs.
- When a change affects agent workflow, extraction behavior, schema boundaries, pipeline contracts, or user-facing operating guidance, update the relevant `.md` harness files in the same change.
- Common harness files include `.cursor/agents/*.md`, `prompts/**/*.md`, `README.md`, `apps/**/README.md`, and design/spec docs such as `biocontrol_toolkit_design_spec.md`.
- Do not leave prompt, agent, or workflow Markdown describing behavior that the code no longer implements.
- If the correct Markdown update is unclear, add a short `TODO` at the evidence boundary instead of leaving stale guidance behind.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/rfuisz) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
