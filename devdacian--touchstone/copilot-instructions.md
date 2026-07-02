## touchstone

> This repo develops Touchstone and dogfoods it on itself; the methodology lives

# Codex entry (Touchstone source repo)

This repo develops Touchstone and dogfoods it on itself; the methodology lives
under `.touchstone/methodology/`.

Before any skill / methodology / Touchstone / consult / review-loop / external-review
work, FIRST read `.touchstone/methodology/TOUCHSTONE.md` — the COMPLETE runtime-neutral
process, self-sufficient for a Codex orchestrator (it includes the pinned reverse
cross-model wrapper for running a Claude reviewer). Under Claude Code, ALSO read
`.touchstone/methodology/TOUCHSTONE-claude.md` for the Claude-specific mechanisms.

This repo's own dogfood tests for the external-review machinery live in
`tests/external-review/` (NOT under `.touchstone/methodology/`, so they are never
vendored into hosts). When you change the classifier or wrappers under
`.touchstone/methodology/scripts/external-review/`, run and keep them green:
`bash tests/external-review/test-classifier.sh` and
`bash tests/external-review/test-forward-wrapper-argv.sh` and
`bash tests/external-review/test-reverse-wrapper-argv.sh`.

When you change `install.sh`, run and keep green:
`bash -n install.sh` and
`bash tests/test-install-output.sh`.

---
> Source: [devdacian/touchstone](https://github.com/devdacian/touchstone) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-01 -->
