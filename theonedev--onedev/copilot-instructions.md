## onedev

> Do not add Co-authored-by trailers when generating commit messages


# No Co-Author Trailers in Commit Messages

**Note:** Do not add co-author or AI attribution info to commit messages in
future work unless the user explicitly asks for it.

When drafting, generating, or composing a git commit message (including via
`generate-commit-message` or before `git commit`):

- Do **not** append `Co-authored-by:` lines or other co-author / AI attribution
  footers to the message.
- Do **not** add `Co-authored-by: Cursor` or similar agent attribution.

Include only subject, body, and footers required by the project's commit-message
requirement (e.g. `Fixes #123`, breaking-change notices). Add co-author trailers
only if the user explicitly asks for them or a non-empty requirement from
`tod get-commit-message-requirement` mandates them.

---
> Source: [theonedev/onedev](https://github.com/theonedev/onedev) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-26 -->
