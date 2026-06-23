## ai-native-eval

> `ai-native-eval` is an agent skill repository for evaluating whether another repository can support repeatable, reviewable AI-assisted development. The repo must dogfood that same standard: changes should be understandable through documented commands, skill contracts, tests, review evidence, and persisted level artifacts rather than chat memory alone.

# Agent Guide

## Mission

`ai-native-eval` is an agent skill repository for evaluating whether another repository can support repeatable, reviewable AI-assisted development. The repo must dogfood that same standard: changes should be understandable through documented commands, skill contracts, tests, review evidence, and persisted level artifacts rather than chat memory alone.

## Current Scope

The repository currently owns:

- Codex agent skills under `.agents/skills/**`.
- The deterministic TypeScript eval tool under `.agents/skills/ai-native-eval/scripts/eval/`.
- Skill eval fixtures and wrappers.
- Self-evaluation artifacts under `.ai-native-eval/` when a run is intentionally committed.

Do not add product-domain application code, hosted service code, database code, or unrelated workflow automation unless the user explicitly expands scope.

## Required Reading

Before changing behavior, read the relevant files:

- [README.md](README.md)
- [docs/runtime.md](docs/runtime.md)
- [docs/architecture.md](docs/architecture.md)
- [docs/reviewer-contract.md](docs/reviewer-contract.md)
- [docs/evidence.md](docs/evidence.md)
- [docs/foundation-self-evaluation.md](docs/foundation-self-evaluation.md)
- [.agents/skills/ai-native-eval/SKILL.md](.agents/skills/ai-native-eval/SKILL.md)

When changing a specific evaluator, read that evaluator's own `SKILL.md` and only its direct child evaluators as needed. Do not create a central canonical evaluator tree.

## Runtime And Test Commands

Use project-owned scripts from the repository root:

```sh
pnpm install --frozen-lockfile
pnpm build
pnpm test
pnpm render:example
pnpm score:example
pnpm self-eval:validate
pnpm self-eval:render
pnpm self-eval:check
pnpm skill-eval:contract
```

`pnpm skill-eval:live` and the eval tool's real-agent tests intentionally invoke a real Codex agent. Run them when the task is about real agent behavior or when the user asks for live evaluation evidence.

## Worktree And Branch Safety

- Never push directly to `main`; publish changes through a pull request.
- Create work branches with the `codex/` prefix unless the user asks otherwise.
- Check `git status --short --branch` before editing, before committing, and before opening a PR.
- Do not revert user changes. If an unrelated dirty file exists, leave it alone.
- For parallel agent work, use separate branches or worktrees per task and record which branch owns which change.
- Clean up temporary run folders, screenshots, and generated logs unless they are deliberate review artifacts.

## Parallel Agent Capacity

Before planning fan-out work, compare the intended number of workers, reviewers, repair loops, and browser/test workers against local Codex capacity. The recommended local setting is:

```toml
[agents]
max_threads = 10
```

Prefer a single execution batch for normal evaluator work. Fan out only when evidence size, independent judgment, risk, or explicit user direction justifies it.

## Review Contract

Every PR should make these points explicit:

- What changed.
- What is intentionally not done.
- Which command evidence ran.
- Which skill contracts or evaluator rubrics changed.
- Which self-evaluation artifact changed, if any.
- Whether live agent evals were run or intentionally deferred.
- Any human-only review decisions.
- Any skipped gate and the reason it is safe.

P0 findings must be fixed before merge. P1/P2 findings need either a fix, a documented follow-up, or an explicit owner decision.

## Thread Closeout

Final answers after substantive work should summarize this thread's remaining work only. If nothing remains in this thread, say so directly. If useful unblocked work remains, include one copy-pastable continuation prompt.

---
> Source: [aiaccelerationism/ai-native-eval](https://github.com/aiaccelerationism/ai-native-eval) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-23 -->
