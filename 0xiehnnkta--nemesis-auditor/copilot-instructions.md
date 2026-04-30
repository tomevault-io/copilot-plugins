## nemesis-auditor

> **NEMESIS** is an iterative deep-logic security audit agent for [Claude Code](https://docs.anthropic.com/en/docs/claude-code). It orchestrates two sub-agents in an alternating feedback loop to find bugs that neither can catch alone. Language-agnostic -- works on Solidity, Move, Rust, Go, C++, Python, TypeScript, or any codebase.

# Nemesis Agent

## What This Is

**NEMESIS** is an iterative deep-logic security audit agent for [Claude Code](https://docs.anthropic.com/en/docs/claude-code). It orchestrates two sub-agents in an alternating feedback loop to find bugs that neither can catch alone. Language-agnostic -- works on Solidity, Move, Rust, Go, C++, Python, TypeScript, or any codebase.

## How to Use

1. Copy the `.claude/` folder into your target project (or run Claude Code in this directory)
2. Start Claude Code: `claude`
3. Type `/nemesis` to launch the full iterative audit

## Available Commands

| Command | Action |
|---------|--------|
| `/nemesis` | Full Nemesis audit (iterative back-and-forth until convergence) |
| `/nemesis --pass1` | Pass 1 only -- run full Feynman Auditor |
| `/nemesis --pass2` | Pass 2 only -- run full State Inconsistency Auditor on existing Pass 1 output |
| `/nemesis --continue` | Continue iterating from where the last pass left off |
| `/nemesis --contract [name]` | Full Nemesis audit on a single contract |
| `/feynman` | Run the Feynman Auditor standalone |
| `/state-audit` | Run the State Inconsistency Auditor standalone |

## How It Works

```
PASS 1: Feynman Auditor (full run)
  Questions every line. Exposes assumptions. Flags suspects.

        | feed forward |

PASS 2: State Inconsistency Auditor (full run, enriched by Pass 1)
  Maps coupled state. Finds mutation gaps. Uses Feynman suspects as targets.

        | feed forward |

PASS 3+: Alternating targeted passes until convergence
  Each pass interrogates the previous pass's new findings.
  Max 6 passes. Nothing survives.
```

## Agent Composition

When `/nemesis` is triggered, the nemesis-auditor orchestrator automatically loads and runs:

1. **Feynman Auditor** (`.claude/skills/feynman-auditor/SKILL.md`) -- Deep business logic bug finder using the Feynman technique. Questions every line, every ordering choice, every guard presence/absence.

2. **State Inconsistency Auditor** (`.claude/skills/state-inconsistency-auditor/SKILL.md`) -- Maps every coupled state pair, every mutation path, and every gap where one side updates without the other.

3. **Iterative Fusion** -- Alternates between the two until no new findings emerge.

## Output

- Per-pass findings: `.audit/findings/feynman-pass[N].md` and `.audit/findings/state-pass[N].md`
- Final consolidated: `.audit/findings/nemesis-verified.md`
- Each finding tagged with discovery path (Feynman-only / State-only / Cross-feed)

---
> Source: [0xiehnnkta/nemesis-auditor](https://github.com/0xiehnnkta/nemesis-auditor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-20 -->
