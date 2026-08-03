## clade

> The operating manual for this repo is [CLAUDE.md](CLAUDE.md). It is written

# Clade — agent operating manual

The operating manual for this repo is [CLAUDE.md](CLAUDE.md). It is written
with Claude Code as the reference operator, but it is agent-agnostic in
substance: whichever coding agent you are, follow it as your instructions.

Two things to know if you are not Claude Code:

1. The enrichment scripts (`scripts/enrich-batch.mjs`, `scripts/cue-tag.mjs`)
   default to calling `claude -p`. If `claude` isn't installed, set
   `CLADE_AGENT_CMD` to your own web-capable backend first — contract and
   examples in [docs/byo-model.md](docs/byo-model.md). You can also do the
   research yourself in-session and write batch files in the schema from
   `docs/schema.md`; the pipeline can't tell the difference.
2. The privacy rules at the top of CLAUDE.md are non-negotiable regardless of
   which agent is operating.

---
> Source: [PropterMalone/clade](https://github.com/PropterMalone/clade) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-25 -->
