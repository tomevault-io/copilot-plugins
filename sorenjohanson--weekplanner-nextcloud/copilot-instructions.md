## weekplanner-nextcloud

> - Think before acting. Read existing files before writing code.

## Approach
- Think before acting. Read existing files before writing code.
- Be concise in output but thorough in reasoning.
- Prefer editing over rewriting whole files.
- Do not re-read files you have already read.
- Test your code before declaring done.
- No sycophantic openers or closing fluff.
- Keep solutions simple and direct. No over-engineering.
- If unsure: say so. Never guess or invent file paths.
- User instructions always override this file.

## Efficiency
- Read before writing. Understand the problem before coding.
- No redundant file reads. Read each file once.
- One focused coding pass. Avoid write-delete-rewrite cycles.
- Test once, fix if needed, verify once. No unnecessary iterations.
- Budget: 50 tool calls maximum. Work efficiently.

## Style
- No caps for emphasis in comments or commit messages. Use prose. Example: write "these parameters must match" instead of "THESE PARAMETERS MUST MATCH", and "the exact scheme" instead of "the EXACT scheme".
- Acronyms (RSA, PSS, CI, JSON, etc.) and constants (`OPENSSL_ALGO_SHA512`) stay capitalised — that is not emphasis, it is naming.

## Commits
- Every commit you author must include a `Co-authored-by` trailer naming the agent that produced the change. The trailer goes at the bottom of the commit message body, separated by a blank line.
- For Claude (Claude Code, claude.ai, etc.): `Co-authored-by: Claude <noreply@anthropic.com>`
- For other agents (Codex, OpenCode, etc.): use that agent's documented attribution.
- One trailer per agent that worked on the commit; if you collaborated with another agent on the same change, include both.

---
> Source: [sorenjohanson/weekplanner-nextcloud](https://github.com/sorenjohanson/weekplanner-nextcloud) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-11 -->
