## ethglobal-cannes-2026-intelligence-exchange

> - Use `skills/token-reduce/SKILL.md` as the default discovery workflow.

# AI Assistant Rules

## Token Efficiency

- Use `skills/token-reduce/SKILL.md` as the default discovery workflow.
- Start with `./skills/token-reduce/scripts/token-reduce-paths.sh topic words` when location is uncertain.
- Prefer targeted reads and scoped `rg -g`.
- Do not start exploration with `find .`, `ls -R`, `grep -R`, `tree`, or `rg --files .`.

## PR Attribution

- All PRs must include:
  - `**Agent:** <MODEL NAME>`
  - `**Co-authored-by:** Chimera <chimera_defi@protonmail.com>`
  - `## Original Request`
- All commits must include:
  - `[Agent: <MODEL NAME>]` in the title
  - `Co-authored-by: Chimera <chimera_defi@protonmail.com>` in the trailer

## Git Discipline

- Never push directly to `main`.
- One task = one branch = one PR.
- When a task is complete and the user has not opted out, create the branch, commit, push, and open a PR instead of leaving changes uncommitted on `main`.
- Keep related commits stacked in the same PR.

## Multi-Pass Review

Every non-trivial change must pass:
1. Correctness review
2. Consistency review across docs/spec/tests
3. Adversarial review
4. Slop-removal pass

## No AI Slop

- No generic filler.
- No hand-wavy “future work” where a current decision is required.
- No decorative sponsor logos or integrations.
- No overstated autonomy, liquidity, security, or market claims.
- If a claim cannot be demonstrated, label it as planned or non-scope.

## Repo Shape

- `spec/` is the source of truth.
- `skills/token-reduce/` is vendored locally for agent workflows.
- Use `.cursor/artifacts/` for generated working notes if needed.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/chimera-defi) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
