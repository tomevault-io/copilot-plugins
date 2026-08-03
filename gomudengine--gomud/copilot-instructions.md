## gomud

> - Use this file for changes under `.github/`, including workflows, custom composite actions, release docs, and local `act` fixtures.

# GitHub Automation Guide

## Scope

- Use this file for changes under `.github/`, including workflows, custom composite actions, release docs, and local `act` fixtures.
- Keep workflow edits tightly scoped to the named workflow or action. Do not change adjacent automation unless the task requires it.

## Working Rules

- Prefer repo-native validation through `make ci-local` when workflow files change.
- Keep trigger shapes intentional. This repo currently uses pull request triggers such as `opened`, `synchronize`, `reopened`, and `ready_for_review` where needed.
- Preserve `persist-credentials: false` on checkout steps unless there is a concrete reason to change authentication behavior.
- Reuse local composite actions in `.github/actions/` when they already capture setup shared across workflows.
- Treat release behavior as split by stability: pushes to `master` update the rolling prerelease, and manual `workflow_dispatch` runs create protected stable releases as documented in `.github/RELEASING.md`.

## Verification

- For workflow or action changes, run `make ci-local` when practical.
- If `make ci-local` is too broad for the task, at minimum run the relevant local dry run or lint path and say what was not run.
- Keep PR descriptions and review notes explicit about which workflow files changed and which checks were actually executed.

---
> Source: [GoMudEngine/GoMud](https://github.com/GoMudEngine/GoMud) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-22 -->
