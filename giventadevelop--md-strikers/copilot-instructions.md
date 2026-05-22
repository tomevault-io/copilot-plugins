## git-local-push

> Standard procedure for using local Git (CLI) to stage, commit, and push changes from Cursor on Windows PowerShell


- **Scope**
  - Applies when committing code changes from the local workspace using Git CLI on Windows PowerShell.
  - Use for routine commits and pushes to existing or new branches.

- **Pre-checks**
  - **Verify current branch**:
    ```powershell
git status -sb
    ```
  - If you need a new branch:
    ```powershell
git checkout -b <new-branch-name>
    ```

- **Stage → Commit → Push (Windows PowerShell)**
  - PowerShell does not support `&&` as a command separator by default. Run each command on its own line.
  - **Stage files**:
    ```powershell
git add <path1> <path2> <pathN>
    ```
  - **Commit with message**:
    ```powershell
git commit -m "<type(scope): short, imperative message>"
    ```
  - **Push to upstream (existing tracking branch)**:
    ```powershell
git push
    ```
  - **First push for a new branch (set upstream tracking)**:
    ```powershell
git push -u origin <branch-name>
    ```

- **Examples**
  - Commit a feature on the current branch:
    ```powershell
git add src/app/api/stripe/payment-intent/route.ts src/components/StripePaymentRequestButton.tsx
git commit -m "feat(payments): add Payment Request Button and PI endpoint"
git push
    ```
  - Create and push a new branch:
    ```powershell
git checkout -b payment-options
git push -u origin payment-options
    ```

- **Authentication prompts**
  - If Git asks to authenticate, a browser window may open. Complete auth and re-run `git push` if needed.

- **Troubleshooting**
  - If you see `The token '&&' is not a valid statement separator`, split combined commands into separate lines.
  - Normalize line endings warning (LF→CRLF) is informational. Commit proceeds unless you configured otherwise.

- **Best practices**
  - Use conventional commit messages when possible (e.g., `feat`, `fix`, `chore`).
  - Keep commits focused and scoped; avoid bundling unrelated changes.
  - Push to a feature branch, then open a PR on GitHub.

- **References**
  - After adding or updating rules, verify consistency with [cursor_rules.mdc](mdc:.cursor/rules/cursor_rules.mdc).

---
> Source: [giventadevelop/md-strikers](https://github.com/giventadevelop/md-strikers) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
