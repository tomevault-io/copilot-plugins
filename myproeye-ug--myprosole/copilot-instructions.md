## git-workflow

> Git and version-control workflow for the MyProSole repo


# GitHub / Version-Control Workflow

Keep work recoverable and the history clean.

- After completing any meaningful unit of work, commit with a clear message and push to `origin` so nothing is lost.
- Stage specific paths with `git add <path>`. Never use blanket adds (`git add -A` / `git add .`) that could sweep in secrets or local data.
- Before every commit, verify nothing sensitive is staged: `git diff --cached --name-only`, then check against `sk-proj|n8n key|secret|credential|token|.env|.pem`.
- Write focused, descriptive commit messages (what changed and why). One logical change per commit.
- Push with a normal `git push origin main`. NEVER force-push to `main`.
- Never rewrite published history (rebase/amend/force) without explicit user approval.
- A post-commit hook may auto-push; still run an explicit `git push origin main` and confirm `git status -sb` shows `## main...origin/main` with nothing ahead.

Windows / PowerShell 5.1 notes:

- `&&` is NOT supported — chain commands with `;`.
- `rg` may be unavailable — use `git ... | Select-String <pattern>`.
- For multi-line commit messages, write a temp file and `git commit -F <file>`, then delete the temp file.

---
> Source: [MyProEye-UG/MyProSole](https://github.com/MyProEye-UG/MyProSole) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-17 -->
