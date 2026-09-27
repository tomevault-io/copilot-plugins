## paseo-agy-acp

> Never let Cursor Co-authored-by reach git history or GitHub Contributors


# No Cursor git co-author

Commits on this repo are authored only by `tiezbro`.

After every `git commit`, run `git log -1 --format=%B`. If you see `Co-authored-by: Cursor` or `cursoragent@cursor.com`, do **not** push. Strip the trailer with `git commit-tree` as documented in `agent.md`, then confirm the message is clean.

Never add a `Co-authored-by` line for Cursor, Cursor Agent, or cursoragent.

If a Cursor-trailer commit already reached `main`: rewrite it and force-push **origin** (`git push --force-with-lease origin main`). Never toggle GitHub visibility to hide contributors (that deletes stars and forks).

Push `origin` (GitHub). Forgejo is the read-only pull mirror.

---
> Source: [tiezbro/paseo-agy-acp](https://github.com/tiezbro/paseo-agy-acp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-27 -->
