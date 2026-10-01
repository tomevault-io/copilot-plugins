## badminton-store

> These rules apply to every coding agent working in this repository.

# Badminton Store agent rules

These rules apply to every coding agent working in this repository.

## Architecture

- Preserve the feature-first Flutter structure, MVVM boundaries, Riverpod dependency injection, and GoRouter navigation already used by the project.
- Treat `supabase/migrations/` as the database source of truth.
- Never edit a migration that has already been deployed. Create a new migration with the Supabase CLI when a task explicitly requires a schema change.
- Keep changes inside the active task's scope. Do not perform opportunistic refactors.

## Security

- Never read, print, stage, or commit `.env` files, credentials, tokens, private keys, or Supabase secret/service-role keys.
- A Flutter client may contain only a Supabase publishable key (or legacy anon key for compatibility).
- Enable and verify RLS for every table exposed through the Supabase Data API.
- Do not use user-editable metadata for authorization.
- Do not weaken RLS, tests, lint rules, or validation to make a task pass.
- Checkout totals, inventory changes, payment state, and privileged order transitions must be validated server-side.

## Git ownership

- Codex is the technical lead: it creates the task branch from `develop`,
  reviews the pull request, posts requested changes, and performs the final
  merge into `develop` without confirmation.
- Cursor is the implementation developer. On the controller-created task
  branch only, Cursor may stage task-scoped files, commit, push that branch,
  open a pull request targeting `develop`, and push follow-up review fixes.
- Cursor must never push directly to `develop`, merge or close a pull request,
  change branches, alter remotes, bypass branch protection, or approve its own
  pull request.
- Never force-push or run destructive Git commands.

## Verification

- Format changed Dart files.
- Run `flutter analyze` and relevant tests.
- Add tests for changed behavior.
- For Supabase changes, run the relevant database/RLS tests and security checks.
- Report blockers honestly; do not claim a check passed unless it was run successfully.

---
> Source: [khanhvo1104/Badminton-Store](https://github.com/khanhvo1104/Badminton-Store) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-30 -->
