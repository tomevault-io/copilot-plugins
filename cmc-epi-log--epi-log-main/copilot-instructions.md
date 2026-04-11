## epi-log-main

> Repository `CMC-EPI-LOG/EPI-LOG-MAIN` has an active branch ruleset:

# AGENTS.md

## Main Merge Ruleset (GitHub)

Repository `CMC-EPI-LOG/EPI-LOG-MAIN` has an active branch ruleset:

- Name: `Protect main branch`
- Ruleset ID: `13352882`
- Enforcement: `active`
- Targets: `~DEFAULT_BRANCH`, `refs/heads/main`

### Active requirements for `main`

- Direct destructive updates to `main` are blocked.
  - `deletion` rule enabled
  - `non_fast_forward` rule enabled (force-push blocked)
- Merge must go through Pull Request (`pull_request` rule).
  - Required approvals: `1`
  - Dismiss stale reviews on new commits: `true`
  - Require conversation resolution before merging: `true`
  - Allowed merge methods: `merge`, `squash`, `rebase`
- Required status checks (`required_status_checks` rule) with strict policy enabled.
  - `Vercel`
  - `Vercel Preview Comments`

## Agent merge protocol (must follow)

When work needs to land on `main`, always:

1. Work on a feature branch (prefer `codex/*`), never commit directly to `main` for merge work.
2. Open PR to `main`.
3. Wait until required checks pass and PR is up to date with base (strict checks policy).
4. Ensure at least one approving review exists.
5. If new commits are pushed after approval, get approval again (stale approvals are dismissed).
6. Ensure no unresolved review conversation remains before merging.
7. Merge using only allowed methods (`merge`, `squash`, or `rebase`) as requested by the user.
8. Never force-push or delete `main`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/CMC-EPI-LOG)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/CMC-EPI-LOG)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
