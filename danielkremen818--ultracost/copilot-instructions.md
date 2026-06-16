## ultracost

> Onboarding for an AI agent (or human) working **on the ultracost repository**.

# AGENTS.md

Onboarding for an AI agent (or human) working **on the ultracost repository**.

> This file is about *developing ultracost*. It is **distinct** from the routing
> block ultracost injects into a *user's* `~/.claude/CLAUDE.md` (the generated
> `<!-- ultracost:start -->` … `<!-- ultracost:end -->` section compiled from
> `policy.json`). Do not confuse the two.

## Purpose

ultracost keeps Claude Code's `ultracode` dynamic workflows from silently
running every `agent()` subagent on Opus: it injects an explicit per-stage
routing policy each session and ships a static guard that fails any unpinned
stage.

## Structure map

One shared core, two delivery surfaces, both compiled from the same
`policy.json`:

```
src/            shared core
  policy.js     load + normalize policy.json, classify/resolve models
  rules.js      compile the policy into the CLAUDE.md routing block
  guard.js      static analysis of agent() stages (UC001–UC005), --fix
  estimate.js   static cost estimate (model mix, tiered vs all-opus)
  pricing.js    parse + refresh the official rate table
  install.js    init/uninstall (writes under ~/.claude, honors CLAUDE_CONFIG_DIR)
  paths.js, log.js, index.js
bin/cli.js      npm CLI surface — init · check · audit · estimate · pricing · status · doctor · uninstall
skills/         plugin surface — routing-policy skill
commands/       plugin surface — /ultracost:check
hooks/          plugin surface — hooks.json (SessionStart injection)
templates/      installed artifacts: reinject.mjs, workflow-gate.mjs, policy.default.json
.claude-plugin/ plugin.json + marketplace.json
tests/          node:test suites (guard, estimate, hook)
examples/       workflow.good.js — dogfood fixture the guard must keep passing
docs/           architecture, policy, estimates, ultracode, testing, publishing
```

## Where the policy lives

`policy.json` is the **source of truth**. The default ships as
`templates/policy.default.json`; `src/policy.js` loads/normalizes it and
`src/rules.js` compiles it into the injected CLAUDE.md block. Change the policy
or the compiler — **never hand-edit a generated `ultracost:start` block.**

## Commands

```bash
npm test                                          # full node:test suite
node bin/cli.js check examples/workflow.good.js   # the dogfood guard run
CLAUDE_CONFIG_DIR=$(mktemp -d) node bin/cli.js init   # sandboxed install test
```

## Invariants (keep green)

1. `npm test` passes (CI matrix: Node 24 / 26).
2. `node bin/cli.js check examples/workflow.good.js` exits 0 (the guard stays
   green on the dogfood fixture).

Both are enforced in CI; a change that breaks either is not merge-ready.

## Scope

Claude Code **only** — the `ultracode` / dynamic-workflow / `agent()` model and
Opus `@ xhigh` don't exist on other harnesses. Do **not** add multi-harness
packaging (`.codex-plugin/`, `.cursor-plugin/`, `.opencode/`, `GEMINI.md`,
etc.); zero runtime dependencies is a hard constraint.

See [CONTRIBUTING.md](CONTRIBUTING.md) for the PR workflow and
[docs/architecture.md](docs/architecture.md) for the full design.

---
> Source: [danielkremen818/ultracost](https://github.com/danielkremen818/ultracost) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-16 -->
