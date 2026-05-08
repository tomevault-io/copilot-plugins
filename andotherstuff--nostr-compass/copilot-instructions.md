## nostr-compass

> When working on newsletter updates:

# Nostr Compass Newsletter Guidelines

## PR Management for Newsletters

When working on newsletter updates:

1. **ALWAYS check for existing PR first** - Newsletter branches typically have an open PR already
2. **Update existing PR** - Force push changes to the existing newsletter branch
3. **Never create duplicate PRs** - One PR per newsletter issue

### Workflow
```bash
# Check for existing PR
gh pr list --head newsletter/YYYY-MM-DD

# If PR exists, update it
git checkout newsletter/YYYY-MM-DD
# Make changes
git add .
git commit --amend
git push --force

# If no PR exists (rare), create one
gh pr create --title "Newsletter #X (YYYY-MM-DD)" ...
```

## Common Newsletter Fixes

- Link updates (njump to GitHub releases, broken URLs)
- Typo corrections
- Content accuracy updates
- Translation synchronization

---
> Source: [andotherstuff/nostr-compass](https://github.com/andotherstuff/nostr-compass) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-24 -->
