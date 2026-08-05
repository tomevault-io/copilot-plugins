## omegaos

> OmegaOS is an open-source agentic terminal operating system built on **rmux** (Rust terminal multiplexer).

# OmegaOS — Agentic Terminal Operating System

## What is this?

OmegaOS is an open-source agentic terminal operating system built on **rmux** (Rust terminal multiplexer).
It turns any VPS or local machine into a fully autonomous multi-agent development platform.

Anyone clones this repo, runs `./install.sh`, and gets:
- A daemon-backed terminal multiplexer (rmux) with typed SDK
- An AI agent orchestration layer (oracle/worker hierarchy)
- A session manager with live progress tracking (TUI menu)
- Quality gates, audit chains, and adversarial verification
- Telegram/webhook integration for human-in-the-loop

## Architecture — 4 Levels

```
Level 1 — Human Interface (Telegram / CLI / Web)
    ↓ intent
Level 2 — AISB Orchestrator (persistent daemon)
    ↓ dispatch
Level 3 — Oracle (1 per project, strategic)
    ↓ decompose + delegate
Level 4 — Workers (ephemeral, parallel, file-lock scoped)
    ↓ execute → verify → report
```

## Tech Stack

- **rmux** — Rust terminal multiplexer (daemon, SDK, hooks, PTY)
- **Claude Code CLI** — AI agent runtime (or any CLI agent)
- **Convex** (optional) — Real-time backend for state sync
- **Hermes** (optional) — Multi-agent coordinator via Gemini

## Key Patterns

### Inspired by tmux-claude UX (re-implemented in Rust against the rmux SDK — no tmux runtime dependency)
- Option+Z session manager menu (fzf-based, grouped, progress bars)
- Oracle/worker tree hierarchy display
- Team spawn + layout (N agents in split panes)
- Session protection, kill history, auto-discovery

### From Omega System (Orchestration)
- 14 AISB Matrix agents (Oracle, Morpheus, Seraph, Keymaker, etc.)
- Quality gates (rubric, grader consensus, Popper falsification)
- Done.json webhook → Telegram reports
- Multi-account Claude rotation for unlimited budget

### From earendil/coding-agent (Session Architecture)
- Service-Session-Runtime separation
- RPC mode for external orchestration
- JSONL session persistence with branching
- Extension hooks for customization

## Laws vs Rules

- **Laws (L0–L6)** — inviolable, universal, top-priority. They bind every agent always and
  override every rule and task. Few, stable, never scoped-out. Rendered first everywhere
  (TUI System tab, `omega rules list`, every prompt block) and visually distinct.
  The seven: L0 ship-the-truth (install-parity), L1 runtime-is-truth, L2 researcher-not-sycophant,
  L3 decide-and-proceed, L4 done-means-100%, L5 quality-over-speed, L6 finish-the-mission
  (never stop mid-workflow — enumerate, execute, verify, report; three legal stops only).
- **Rules (R-*, named)** — operational, categorized (Universal / QualityGate / Orchestration /
  Reporting / Safety), scoped per agent level (Master / Global / Oracle / Worker via the explicit
  `scopes` field). Guidelines that implement the Laws in practice.

Source of truth: `crates/omega-core/src/rules.rs` (`RuleKind::{Law, Rule}`).

Delivery: every dispatched agent (Oracle/Worker), on any LLM backend, receives its role-scoped Laws+Rules via the single funnel `rules::agent_context_block(scope)`. The Master gets them via its runtime prompt (`_master-runtime.md`). Provider-agnostic — the context is plain text injected into the prompt.

## Development Rules

- **LAW 0 — INSTALL PARITY (NON-NEGOTIABLE): every improvement to OmegaOS MUST
  keep `install.sh` complete. A feature is NOT done until a fresh
  `git clone … && ./install.sh` reproduces it.** Before declaring any change
  done:
  1. New asset (agent/command/config/template/cron/dir)? → add the copy/setup
     step to `install.sh` (binary changes ship automatically — `install.sh`
     builds from source).
  2. Run `./scripts/verify-install.sh` — it must pass (binary-from-source,
     agents, commands, configs, crons, **no secrets tracked**, git clean,
     remote in sync).
  3. `git add -A && commit && push` — GitHub always holds the latest, and the
     installer always installs the latest. NEVER leave an improvement that a
     fresh install wouldn't get.
  Secrets (tokens, creds) live in `~/.omega/` only — gitignored, NEVER in the repo.
- Law 1: Code lies. Only runtime tells the truth.
- Law 2: Researcher, not sycophant. Challenge flawed premises.
- Every feature must be verified with live runtime evidence before merge.
- Commits must pass build + lint + typecheck.
- No --force, no --no-verify, no secrets in code.

<!-- AGENTIK-LAYOUT:START — convention de placement des fichiers (gérée par /project-tidy) -->
## 📁 Où écrire les fichiers (convention — À RESPECTER)

Pour garder la codebase propre, chaque fichier créé va à un endroit précis :

- **Code** → dossiers de code existants (`app/`, `components/`, `lib/`, `convex/`, `services/`…). Jamais de code à la racine.
- **Doc humaine** (specs, guides, knowledge, notes de conception) → **`docs/`**.
- **Sorties/tracking d'agents VISIBLES** (rapports d'audit, tests jetables, logs, captures, plans exportés, fourre-tout) → **`agentic/`** :
  `agentic/audits/` · `agentic/reports/` · `agentic/tests/` · `agentic/specs/` · `agentic/archive/`.
- **Système OmegaOS** (`.planner/`, `.audit/`, `.oracles/`) → **restent à la racine** (gérés par OmegaOS, ne pas déplacer).
- **Canon** (`README.md`, `CLAUDE.md`, `RULES.md`, `PROGRESS.md`, `vision/`, `PRD…`, `*feature*`, `*step*`) → restent à la racine, ne pas déplacer.

**Interdit** : créer un nouveau dossier de tracking visible à la racine (`audits/`, `report.md`, `to order/`, `deep-test-*.mjs`…). → tout ça va dans `agentic/`.

**Règle d'or** : si ce n'est ni du code, ni du canon, ni un dotfolder système → ça va dans `docs/` (humain) ou `agentic/` (agent).
<!-- AGENTIK-LAYOUT:END -->

---
> Source: [agentik-os/OmegaOS](https://github.com/agentik-os/OmegaOS) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-04 -->
