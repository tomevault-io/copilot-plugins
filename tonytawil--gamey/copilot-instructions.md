## gamey

> - **Work directly on `main` by default** - This repo does not use feature branches or PRs unless the user explicitly asks for them.

# Agent Guidelines

## Workflow

- **Work directly on `main` by default** - This repo does not use feature branches or PRs unless the user explicitly asks for them.
- **Commit and push regularly** - Don't let changes pile up. Commit after completing each feature or fix, then push.
- **Never commit API keys or secrets** - Always use `.env.local` for sensitive data. Verify it's in `.gitignore` before committing.

## UI/UX Preferences

### Don't Do
- **No decorative glow effects** - Keep visuals clean and functional
- **No pointless status indicators** - If it doesn't provide actionable info, remove it
- **No small game canvases** - Games should be prominent and large

### Do
- **Prioritize the game** - The playable area should dominate the screen
- **Keep UI minimal** - Only show information that matters
- **Functional over decorative** - Every element should have a purpose

---
> Source: [TonyTawil/Gamey](https://github.com/TonyTawil/Gamey) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
