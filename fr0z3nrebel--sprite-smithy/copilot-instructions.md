## sprite-smithy

> Guidelines for writing semantic commit messages


# Commit Message Guidelines

When writing commit messages, follow these rules:

## Title

- Use **semantic** style (e.g., `feat:`, `fix:`, `docs:`, `refactor:`, `chore:`)
- Keep the title to **50 characters or less** including spaces
- Use imperative mood (e.g., "Add" not "Added")

## Body

- Wrap lines at **72 characters** including spaces
- The body must have **3 sentences or paragraphs**, each separated by a blank line:

1. **What** — Describe what changed in concrete terms.
2. **Why** — Explain the rationale or motivation for the change.
3. **Testing** — If applicable, describe how the change was tested. If no testing was done, explain why (e.g., config-only change, docs update, etc.).

## Example

```
feat: add dark mode toggle to settings

Add a toggle switch in the settings page that persists the user's
theme preference to localStorage.

Users requested dark mode support to reduce eye strain during
evening use.

Manually verified toggle behavior and persistence across page
reloads; no automated tests added for this UI-only change.
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Fr0z3nRebel) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
