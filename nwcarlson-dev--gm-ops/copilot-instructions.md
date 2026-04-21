## gm-ops

> Always explain shell commands in plain language before running them


# Command Approval Context

When running shell commands that require user approval, always explain what the command does in plain, non-technical language before executing it.

- Describe the **purpose** of the command (what it accomplishes)
- Describe the **effect** (what will change on their system)
- If the command is destructive or irreversible, call that out explicitly
- Never assume the user can read the command syntax to understand what it does

Example:

> **Next I need to upload all your local commits to GitHub, overwriting what's currently there** — this is safe because your local version is the most up-to-date.
>
> `git push --force origin main`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/nwcarlson-dev) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
