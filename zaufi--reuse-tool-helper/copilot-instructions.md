## reuse-tool-helper

> SPDX-FileCopyrightText: 2026 Alex Turbov <i.zaufi@gmail.com>

<!--
SPDX-FileCopyrightText: 2026 Alex Turbov <i.zaufi@gmail.com>
SPDX-License-Identifier: CC0-1.0
-->

# Repository Notes

- Run ShellCheck via `pre-commit run -a shellcheck`.
- `*.md` files must pass `pre-commit run -a markdownlint`.
- Prefer `rg`/`rg --files` for searching.
- Keep edits minimal and avoid reverting unrelated user changes.
- Write commit titles as Conventional Commits and keep them compliant with `.gitlint`:
  use an allowed type, format `<type>: <lowercase subject>`, and keep the title at
  72 characters or fewer.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/zaufi)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/zaufi)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
