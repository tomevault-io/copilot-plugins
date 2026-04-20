## dds-platform

> **You MUST have explicit permission before any git operations:**

# DDS Platform - Claude Configuration

## Git & Commits - CRITICAL RULE

**You MUST have explicit permission before any git operations:**

- ❌ Never commit without asking permission first
- ❌ Never add files to git staging without asking permission
- ❌ Never push to remote without asking permission
- ❌ Never include git operations in a plan without explicit approval

**Process:**
1. Complete the work/testing
2. Show results to user
3. **Ask for explicit permission** to commit/push
4. **Wait for clear approval**
5. Only then execute git operations

This applies whether you're:
- Initiating work in a prompt
- Working during plan mode
- Completing a task
- Testing code

No exceptions. Always ask first.

## Project Structure

- `/apps/blackdot-dev` - Main showcase app
- `/packages/renderer` - DDS renderer with theme system
- `/e2e` - Playwright tests
- `playwright.config.ts` - E2E test configuration

## Testing

Run E2E tests:
```bash
pnpm test:e2e
```

All tests must pass before any commits.

## Theme System

9 theme variants available:
- minimal, vibrant, neon, arctic, sunset, forest, midnight, mist, monochrome

Configured via `data-theme-variant` attribute on `<html>` element.

## Pages

- `/` - Home page with navigation
- `/sections` - All sections from site.config.json
- `/demo` - Theme variants list

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/michaeldouglas319) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
