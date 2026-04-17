## tron

> Always commit and push fixes to GitHub


# GitHub push workflow

- After each code fix/change, **create a git commit and push it to GitHub**.
- Use the SSH remote `tron` (`git@github.com:vrushabd/tron.git`) and push to the `main` branch.
- Do **not** force-push.
- Do **not** commit secrets (e.g. `.env.local`, private keys, tokens). If a fix requires changing secret values, leave them uncommitted and tell the user what to set in GitHub/Vercel env vars instead.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/vrushabd) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
