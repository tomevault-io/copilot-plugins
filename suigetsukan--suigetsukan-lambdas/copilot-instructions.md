## suigetsukan-lambdas

> When user asks to commit/push, use feat: prefix so releases propagate


# Commit and push

When the user asks to **commit** and/or **push** (e.g. "commit this", "commit and push", "commit/push"):

1. **Assume the change is a feature.** Use the conventional commit prefix **`feat:`** in the commit message (e.g. `feat: add X` or `feat(scope): add X`) so that semantic-release in this repo triggers a release and downstream propagation (see `.github/workflows/README.md`).
2. **Set the commit message accordingly.** Derive a clear, concise subject from the staged or changed files; do not use generic messages like "Update files" unless the user provided no context.
3. Proceed with `git add`, `git commit -m "feat: ..."`, and `git push` (pull --rebase first if the remote is ahead).

Only use a different prefix (e.g. `fix:`, `docs:`) if the user explicitly says so or the change is unambiguously a fix/docs-only change.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/suigetsukan) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
