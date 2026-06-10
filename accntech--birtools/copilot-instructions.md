## birtools

> - Never auto-commit. Even when told to "commit changes", surface the proposed grouping and

# Repository Instructions

## Git
- Never auto-commit. Even when told to "commit changes", surface the proposed grouping and
  messages, then wait for approval before running `git commit`. Staging is fine.
- Commit subject must be 65 characters or fewer. If more detail is needed, put it in the body
  after a blank line.
- Use Conventional Commits format: `<type>(<scope>): <description>`.
- Commit type must be lowercase: `feat`, `fix`, `chore`, `refactor`, `docs`, `style`, `test`,
  `perf`, `build`, `ci`, or `revert`.
- Scope is optional but preferred, for example `pdf` or `tauri`.
- Description must be lowercase imperative, for example `add`, `fix`, or `extract`; do not use
  past-tense forms like `added` or third-person forms like `adds`.
- Do not end commit subjects with a period.
- For breaking changes, use `!` before the colon, for example `feat(api)!: update contract`, and
  add a `BREAKING CHANGE:` footer.
- Run `bun run check` before every commit. Block on any error and surface failures instead of
  committing broken code.
- In a worktree, defer commits until implementation is complete. Do not create per-task commits
  during work; surface the full grouping at the end.
- Warn before committing throwaway files such as seed scripts, one-off migration helpers, ad-hoc
  debug scripts, scratch fixtures, or similar single-use artifacts. Recommend deleting or
  ignoring them unless the user explicitly confirms they should be committed.
- When merging worktree branches into `main`, keep history linear and author-date-ordered. Use
  fast-forward merges or cherry-picks, not merge commits.

## Code Style
- Use kebab-case for TypeScript and Svelte filenames.

---
> Source: [accntech/birtools](https://github.com/accntech/birtools) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-10 -->
