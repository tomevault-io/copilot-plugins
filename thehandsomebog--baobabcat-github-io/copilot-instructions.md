## baobabcat-github-io

> This project uses **bd** (beads) for issue tracking. Run `bd onboard` to get started.

# Agent Instructions

This project uses **bd** (beads) for issue tracking. Run `bd onboard` to get started.

## Quick Reference

```bash
bd ready              # Find available work
bd show <id>          # View issue details
bd update <id> --status in_progress  # Claim work
bd close <id>         # Complete work
bd sync               # Sync with git
```

## Development Commands

```bash
npm test              # Run Playwright tests
npm run test:headed   # Run tests with browser visible
npx serve -l 3000     # Start local dev server
node scripts/fetch-ai-news.js  # Update AI news feed locally
```

## Project-Specific Guidelines

- **Static site** - No build step. Edit HTML/CSS/JS directly.
- **Tests require server** - Playwright config auto-starts `npx serve -l 3000`
- **AI news auto-updates** - GitHub Actions runs hourly; don't manually edit ai-pulse.html feed content
- **Contact form** - Uses Web3Forms; access key is in hidden form field
- **CSS variables** - Use existing variables from top of styles.css for consistent theming

## Landing the Plane (Session Completion)

**When ending a work session**, you MUST complete ALL steps below. Work is NOT complete until `git push` succeeds.

**MANDATORY WORKFLOW:**

1. **File issues for remaining work** - Create issues for anything that needs follow-up
2. **Run quality gates** (if code changed) - Tests, linters, builds
3. **Update issue status** - Close finished work, update in-progress items
4. **PUSH TO REMOTE** - This is MANDATORY:
   ```bash
   git pull --rebase
   bd sync
   git push
   git status  # MUST show "up to date with origin"
   ```
5. **Clean up** - Clear stashes, prune remote branches
6. **Verify** - All changes committed AND pushed
7. **Hand off** - Provide context for next session

**CRITICAL RULES:**
- Work is NOT complete until `git push` succeeds
- NEVER stop before pushing - that leaves work stranded locally
- NEVER say "ready to push when you are" - YOU must push
- If push fails, resolve and retry until it succeeds

---
> Source: [Thehandsomebog/baobabcat.github.io](https://github.com/Thehandsomebog/baobabcat.github.io) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-07 -->
