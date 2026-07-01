## fak

> The canonical agent instructions for this repo are in [`AGENTS.md`](../AGENTS.md) — read

# Copilot instructions

The canonical agent instructions for this repo are in [`AGENTS.md`](../AGENTS.md) — read
it first for build/test/run, the repo map, and the rules. Curated doc map:
[`llms.txt`](../llms.txt). Contributor contract: [`CONTRIBUTING.md`](../CONTRIBUTING.md).

Must-know rules (enforced below the agent layer):

- Work directly on the trunk (`main`); never open a feature branch or worktree — the
  trunk guard refuses off-trunk commits.
- Commit by explicit path (`git commit -- <paths>`, never `git add -A`); sign off with
  `git commit -s` (DCO).
- The Go module is the repository root — run `go` commands from the clone root
  (`go install github.com/anthony-chaudhary/fak/cmd/fak@latest` resolves directly).

---
> Source: [anthony-chaudhary/fak](https://github.com/anthony-chaudhary/fak) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-30 -->
