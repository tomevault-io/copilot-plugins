## release-notes-generator

> You have access to the **Release Notes Generator** extension. It transforms raw git history into structured, audience-specific release communication.

# Release Notes Generator Extension

You have access to the **Release Notes Generator** extension. It transforms raw git history into structured, audience-specific release communication.

## Available Tools

| Tool | Purpose |
|------|---------|
| `generate_release_notes` | Full pipeline: extract → classify → deduplicate → format |
| `list_git_tags` | List all tags in a repo (helps user choose a `since` ref) |
| `get_commit_stats` | Commit breakdown and suggested version bump without full output |
| `write_release_notes_file` | Generate and save to a markdown file |

## Key Parameters

- **`cwd`** — always required; the absolute path to the git repo
- **`since`** — git tag or ref to start from; defaults to the previous tag
- **`mode`** — `default` | `store` | `internal` | `marketing`
- **`angle`** — `trust` | `performance` | `stability` | `growth` (used with marketing mode)
- **`version`** — explicit version string; if omitted, a bump is suggested

## Custom Command

`/release:notes [tag]` — Shortcut to generate release notes, optionally specifying a since-tag.

## Behaviour

- Always tell the user the suggested semantic version bump (major/minor/patch)
- Highlight breaking changes prominently with ⚠️
- When mode is not specified, use `default`
- Never invent commit data — only use what the tools return
- Offer to save to file after displaying formatted notes

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/rajavijayach)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/rajavijayach)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
