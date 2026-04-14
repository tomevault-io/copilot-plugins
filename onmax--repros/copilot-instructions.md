## repros

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Purpose

Repository for bug reproductions across Nuxt ecosystem libraries. Each subdirectory is a minimal project demonstrating a specific issue.

## Related Libraries

Source code for maintained libraries lives in `~/nuxt/`:
- `~/nuxt/content` - Nuxt Content
- `~/nuxt/hub` - NuxtHub
- etc.

## Skills

Use the `vue`, `nuxt`, and `nuxthub` skills when working in this repo.

---

## Bug Reproduction Workflow

### 1. Folder Naming

- Bug folder: `{library}-{issue-number}` (e.g., `nuxthub-727`)
- Fix folder: `{library}-{issue-number}-fix` (e.g., `nuxthub-727-fix`)
  - Use `-fix` not `-fix` (shorter, avoids StackBlitz cache conflicts)

### 2. Create Bug Reproduction

1. Create minimal project reproducing the issue
2. Include only what's necessary to demonstrate the bug
3. Add `.gitignore` with: `node_modules`, `.data`, `.nuxt`, `.output`
4. Add README.md (see structure below)

### 3. Verify the Bug

Verification can be:
- **Generated files**: Check `.nuxt/`, `dist/`, or build output
- **Runtime behavior**: Run dev server and observe error
- **Test failure**: Write a test that fails (if repo has tests, this is preferred)
- **Type error**: Check IDE/tsc output

Document the verification method in README.

**Note**: If the target repo has existing tests, try to reproduce the bug with a test first. This helps maintainers and ensures the fix is properly validated.

### 4. Create Fix Folder

1. Copy bug folder to `-fix` version (use `-fix` not `-fix` to avoid StackBlitz cache issues)
2. Ensure `package.json` uses npm versions (not `link:` paths from local dev)
3. Create pnpm patch: `pnpm patch {package}`
4. Apply fix to extracted package
5. Commit patch: `pnpm patch-commit {path}`
6. **Fix pnpm patch for StackBlitz** (see below)
7. Verify fix using same method as bug verification

#### pnpm Patch StackBlitz Compatibility

After `pnpm patch-commit`, pnpm creates `pnpm-workspace.yaml` with only `patchedDependencies`. This breaks StackBlitz. Fix it:

1. Delete `pnpm-workspace.yaml`
2. Move config to `package.json` with **version in key**:
```json
{
  "pnpm": {
    "patchedDependencies": {
      "@nuxt/nitro-server@4.3.0": "patches/@nuxt__nitro-server.patch"
    }
  }
}
```
3. Delete `pnpm-lock.yaml` (StackBlitz generates fresh one)
4. Test locally: `rm -rf node_modules && pnpm i && pnpm dev`

### 5. README Structure

Each repro folder README should contain:

```md
# {library}-{issue-number}

Issue: {link-to-github-issue}

## Problem
{Brief description}

## Verify
```bash
pnpm i && pnpm build
```

## Expected
{What should happen}

## Actual
{What actually happens}
```

For `-fix` folders, add:

```md
## Fix
{What the patch changes}
```

### 6. Push to Repository

```bash
cd ~/repros
git add {folder-name} {folder-name}-fix
git commit -m "add {library}-{issue} repro"
git push
```

---

## Pull Request Workflow

### 1. Before Creating PR

- Explore existing PRs/commits to match repo's style
- Check commit message conventions
- Look at PR description format

### 2. Apply Fix to Source

1. Create branch: `fix/{short-description}`
2. Apply changes from patch to actual source files
3. **Match existing code style and patterns** in the project
4. If tests exist, consider adding a test reproducing the bug (helpful but not mandatory)
5. Run tests: ensure all pass
6. Build: ensure it compiles

### 3. PR Body Structure

```md
Closes #{issue-number}

## Summary
{1-2 sentences explaining the bug and fix}

## StackBlitz

| | Link | Expected |
|---|---|---|
| Bug | [{library}-{issue}](https://stackblitz.com/github/onmax/repros/tree/main/{library}-{issue}?startScript=build) | ❌ Build fails |
| Fix | [{library}-{issue}-fix](https://stackblitz.com/github/onmax/repros/tree/main/{library}-{issue}-fix?startScript=build) | ✅ Build succeeds |

## CLI Reproduction

```bash
git clone --depth 1 --filter=blob:none --sparse https://github.com/onmax/repros.git
cd repros && git sparse-checkout set {library}-{issue}
cd {library}-{issue} && pnpm i && pnpm build
```

## Verify fix

```bash
git sparse-checkout add {library}-{issue}-fix
cd ../{library}-{issue}-fix && pnpm i && pnpm build
```

The `-fix` folder uses [pnpm patch](https://pnpm.io/cli/patch) to apply the fix locally.

## Related
- {link-to-original-issue}
```

### 4. Commit Message

- Follow repo conventions (check recent commits)
- Common format: `{type}({scope}): {description}`
- Examples: `fix(db): pass mode to drizzle for mysql`

### 5. After PR Submission

- Wait for CI to pass
- Address any failing checks before requesting review
- If CI fails, fix and push to same branch

---

## Quick Reference Commands

```bash
# Create pnpm patch
pnpm patch {package}
pnpm patch-commit '{extracted-path}'

# Sparse checkout for repro
git clone --depth 1 --filter=blob:none --sparse https://github.com/onmax/repros.git
cd repros && git sparse-checkout set {folder-name}

# StackBlitz URLs (with auto-install and build)
# Bug: https://stackblitz.com/github/onmax/repros/tree/main/{library}-{issue}?startScript=build
# Fix: https://stackblitz.com/github/onmax/repros/tree/main/{library}-{issue}-fix?startScript=build
#
# startScript param runs the script from package.json after install
# Other useful params: terminal=true, file=path/to/file.ts

# Create PR
gh pr create --repo {org}/{repo} --title "{title}" --body "{body}"
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/onmax)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/onmax)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
