## karma

> **[🇬🇧 English (current)](./CLAUDE.md) · [🇨🇳 中文](./CLAUDE.zh.md)**

# karma v2 — Project collaboration charter

**[🇬🇧 English (current)](./CLAUDE.md) · [🇨🇳 中文](./CLAUDE.zh.md)**

## One-line positioning

karma = Keep Agents from forgetting the user's most-valued core directions in long tasks.

## Strict boundaries (non-negotiable)

karma v2 clear boundaries learned from v1's failure (see [karma-v1/ARCHIVE.md](https://github.com/jhaizhou-ops/karma-v1/blob/main/ARCHIVE.md)):

### Things we don't do

- ❌ **No auto-distilling new rules** — User manually maintains core directions, avoiding LLM distillation noise / misalignment
- ❌ **No retrieval / cosine / scene routing** — 5-10 rules all always-on, no selection needed
- ❌ **Don't compete with memory systems** — "Facts about the user" go to Claude Code auto-memory
- ❌ **v0 no LLM** — All engineering (keywords / regex / counting)
- ❌ **No reward / RL / scoring** — Behavioral reminders aren't reward functions

### Strict LLM authorization (if v1+ uses LLM)

Inherits v1 lessons:

- ✅ Local mlx Qwen3.6 — small tasks + ≤4 concurrent
- ✅ OpenRouter qwen3.5-flash-02-23 nothinking — only allowed remote
- ❌ sonnet / claude API — forbidden

## Working principles

### 1. No cheating against current user

karma v2's validation method is author self-use. **Never**, for self-comfort:
- Hardcode author-specific rules into default templates
- Hardcode author's empirically-found violation words into hooks
- Train anything on author's session data

karma's "defaults" must be **cross-user reasonable** (CLAUDE.md / Anthropic best practice level universal).

### 2. Always design from "I am the user" perspective

For every change, ask yourself:
- Can a first-time karma user get started in 5 minutes?
- Is the yaml config straightforward? Do they need docs to understand?
- After hook install, does the first violation detection make the user say "ah, I see"?

If a feature only works for the author → cut it.

### 3. Validation > accuracy numbers

karma v1 obsessed over accuracy numbers (67% inj precision etc.), causing optimization direction drift.

karma v2's validation criterion is **whether the author can describe 3 concrete cases after a week of self-use**:
- "During some long task, I didn't speak up and the Agent reminded me of X by itself"
- "After some compact, the Agent still remembered Y"
- "Once when the Agent wanted to do Z, I saw the ⚠ prompt and corrected"

If after a week, can't describe such cases → karma's hypothesis is wrong, redesign.

## Task management

karma v2 doesn't maintain complex task lists. "One milestone, one PR" thinking.

### Current milestone: M0 skeleton

- Project initialization ✓
- rules.yaml schema
- 2 hook prototypes
- karma CLI skeleton
- 5-10 seed rules

## Commit conventions

Commit messages explain what was done + why. Brief and direct:

```
feat: rules.yaml parser + schema validation

Supports single-line / multi-line preference, violation_keywords array
Schema validates 5-10 rule cap + id uniqueness
```

No complex templates needed. But every commit must:
- Pass lint
- Not introduce new LLM dependencies (v0 stage)
- Stay under ~200 lines (keep changes small and reviewable)

## Reporting style

Completion reports in 1-3 sentences: **what was done + test results + push status**.
Don't:
- List long todos
- Expand diff
- Explain tool mechanics
- Write "next possible things" over 3 lines

## Autonomous execution authorization

Inherits v1 authorization:
- ✅ After completing tasks, directly git add + commit + push
- ✅ Run tests / install dependencies / fix lint
- ✅ Create comments / issues / PRs
- ✅ Delete merged temporary branches

Still need pre-approval:
- `git push --force` / `git reset --hard` of published commits
- gh repo archive / settings modifications
- Data destruction (~/.claude/karma/ clearing)
- Cross-repo changes

## Failure handling

karma v2's first principle is **honesty** — same as v1:

- If hook doesn't take effect, say so plainly, don't fake it
- If violation detection missed a real violation, record it, don't cover up
- If author self-uses for a week with no felt value, plainly say redesign — don't strongarm to maintain sunk cost

## Relationship with v1

v1 archived in [jhaizhou-ops/karma-v1](https://github.com/jhaizhou-ops/karma-v1).
Any reusable v1 assets (SEED 20 rules / qwen3.5 helper / etc.) can be referenced, but **not required** to reuse.

v2 is a cognitive reboot, not a v1 refactor. If some component designs changed, dropping v1's solution is fine.

---
> Source: [jhaizhou-ops/karma](https://github.com/jhaizhou-ops/karma) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-15 -->
