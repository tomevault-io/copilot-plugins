## replication

> This directory is the only canonical source for the Replication desktop app.

# Replication project rules

This directory is the only canonical source for the Replication desktop app.

- Product scope: one uploaded video in, three publishable replica videos out.
- The three allowed transformations are hook rewrite, ending rewrite, and creator identity replacement.
- Do not add market research, account management, analytics, publishing, content libraries, or a timeline editor.
- Production generation must stay behind explicit per-run paid authorization.
- A run can become `succeeded` only after a real MP4 exists and passes video validation.
- Use the scripts in `package.json` for start, test, and build.

---
> Source: [francoeur003/replication](https://github.com/francoeur003/replication) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-31 -->
