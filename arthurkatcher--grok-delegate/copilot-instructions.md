## grok-delegate

> Instructions for coding agents working in **this repository**.

# AGENTS.md — grok-delegate

Instructions for coding agents working in **this repository**.

## What this repo is

Public GitHub marketplace **Grok Build Extras** for Grok Build. One plugin today:

| | |
|--|--|
| **Plugin** | `grok-delegate` |
| **Path** | `plugins/delegate/` |
| **Commands** | `/delegate-claude`, `/delegate-codex` |
| **Runtime** | Node ESM companion under `plugins/delegate/scripts/` |
| **Version** | `0.1.0` (MIT) |

Clients install with:

```bash
grok plugin marketplace add arthurkatcher/grok-delegate
grok plugin install arthurkatcher/grok-delegate#plugins/delegate --trust
```

The `#plugins/delegate` subdir is required — repo root is the **marketplace**, not the plugin package.

## Layout (do not invent a second one)

```text
marketplace.json                 → Grok Build Extras catalog
plugins/delegate/
  .grok-plugin/plugin.json       → plugin manifest
  commands/                      → slash command markdown
  skills/delegate-runtime/       → how Grok must launch the companion
  scripts/delegate-companion.mjs → CLI entry
  scripts/lib/                   → args, policy, engines, stream, jobs, lifecycle
  tests/                         → unit + integration (node --test)
.github/workflows/ci.yml         → npm test + grok plugin validate
SECURITY.md · CHANGELOG.md · CONTRIBUTING.md · README.md
```

## Non-negotiables

1. **Fail closed** — unknown actions/flags exit non-zero; free text after `--` cannot inject flags.
2. **Claude rescue = no Bash by default** — file tools only; shell/git only via explicit `--allowed-tools` or `--yolo`.
3. **Read actions stay RO** — Claude hermetic/`dontAsk` when API key is set; **Max/OAuth without a key auto-drops `--bare`**. Codex OS `read-only` unless user opts out; **auto `--skip-git-repo-check` outside git**.
4. **Preflight** — every non-setup action checks binary + auth; same next steps as `setup` if not ready. Policy conflicts run **before** readiness.
5. **Host launch** — Grok background task must use `timeout: 0` (or very high); default ~20s cancels long runs.
6. **Cancel kills process group** — do not regress `child-lifecycle` / SIGTERM on Grok task cancel.
7. **No secrets in git** — no keys, tokens, job DBs, or real credentialed configs.
8. **Config vs product** — this is a Grok **plugin**, not Meta MCP Manager; do not import meta-manager layout into this tree.

## How to work here

```bash
cd plugins/delegate && npm test
grok plugin validate plugins/delegate   # if grok is on PATH
```

- Prefer small diffs in existing modules over new frameworks.
- User-facing behavior change → update `CHANGELOG.md` **Unreleased**.
- Install path / flags / safety → keep `README.md` honest.
- Host launch contract → keep `skills/delegate-runtime/SKILL.md` accurate (`background: true` + `--wait`, do not poll companion status into chat).

## Out of scope (unless the user expands)

- Flattening upstream MCP tools into Grok’s tool list  
- Multi-tenant SaaS control plane  
- Making Windows a first-class default  
- Logging full argument/result bodies by default  

## Related docs

| Doc | Use |
|-----|-----|
| `README.md` | Product + install |
| `CONTRIBUTING.md` | PR / test expectations |
| `SECURITY.md` | Vulnerability reporting |
| `CHANGELOG.md` | Released behavior |
| `plugins/delegate/skills/delegate-runtime/SKILL.md` | Runtime host contract |

## Checklist before finishing

1. `npm test` green under `plugins/delegate`.
2. No secrets staged.
3. Safety defaults not weakened without an explicit, documented opt-in.
4. CHANGELOG / README touched if user-visible.

---
> Source: [arthurkatcher/grok-delegate](https://github.com/arthurkatcher/grok-delegate) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-14 -->
