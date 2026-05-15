## insforge-mcp

> Insforge MCP Publishing Guide - Instructions for AI assistants on how to publish this package


# Insforge MCP Publishing Guide

## Package Information
- **Package Name**: `@insforge/mcp`
- **Current Version**: Check `package.json`
- **Registry**: npm (public)

## 🚨 IMPORTANT: Two Publishing Workflows

### 1. DEV Publishing (For Testing)
Use this for internal team testing WITHOUT affecting production users.

```bash
# 1. Build the package
npm run build

# 2. Publish to dev tag
npm publish --tag dev --access public
```

**Result**: Team can install with `npm install -g @insforge/mcp@dev`

### 2. PRODUCTION Publishing (Stable Release)
Use this when dev version is tested and ready for all users.

```bash
# 1. Build the package
npm run build

# 2. Bump version (choose one)
npm version patch  # 1.1.2 -> 1.1.3 (bug fixes)
npm version minor  # 1.1.2 -> 1.2.0 (new features)
npm version major  # 1.1.2 -> 2.0.0 (breaking changes)

# 3. Publish to latest tag (production)
npm publish --access public

# 4. Push version commit and tag to git
git push && git push --tags
```

**Result**: Everyone installs with `npm install -g @insforge/mcp`

## Version Conventions

### Dev Versions (Testing)
- Use pre-release versions: `1.1.3-dev.1`, `1.1.3-dev.2`, etc.
- Publish with `--tag dev`
- Example workflow:
  ```bash
  npm version 1.1.3-dev.1
  npm publish --tag dev --access public
  ```

### Production Versions (Stable)
- Use semantic versioning: `1.2.3`
- Publish without tag (defaults to `latest`)
- Always increment from the last production version, NOT from dev versions

## npm Tag System

Think of tags like Git branches:
- `latest` tag = Production (what users get by default)
- `dev` tag = Development (opt-in testing)

**They coexist in the same package!**

```
@insforge/mcp
├── latest → 1.1.2 (production)
└── dev    → 1.1.3-dev.2 (testing)
```

## Integration with Installer

The `@insforge/install` package has a `--dev` flag:

```bash
# Install production version
npx @insforge/install --client cursor --env API_KEY=key

# Install dev version
npx @insforge/install --client cursor --env API_KEY=key --dev
```

When you publish to `dev` tag, team members use `--dev` flag to test.

## Pre-Publishing Checklist

### For Dev Releases
- [ ] Code changes committed
- [ ] `npm run build` succeeds
- [ ] Version follows pattern: `X.Y.Z-dev.N`
- [ ] Use `npm publish --tag dev --access public`

### For Production Releases
- [ ] Dev version tested by team
- [ ] All tests passing
- [ ] `npm run build` succeeds
- [ ] Version bumped with `npm version patch|minor|major`
- [ ] Use `npm publish --access public` (no --tag)
- [ ] Git tags pushed: `git push --tags`

## Common Commands

```bash
# Check current published versions
npm view @insforge/mcp versions

# Check what's on latest tag
npm view @insforge/mcp@latest version

# Check what's on dev tag
npm view @insforge/mcp@dev version

# Test local build before publishing
npm run build
npm link
# Then test with: insforge-mcp --api_key test
```

## Troubleshooting

**Q: I published to dev, but users are getting the old version**
A: Users must explicitly install with `@dev` tag or use installer with `--dev` flag

**Q: How do I unpublish a dev version?**
A: You can't unpublish (npm policy), but you can publish a new dev version to override

**Q: Should I increment version for every dev publish?**
A: Yes, use `1.1.3-dev.1`, `1.1.3-dev.2`, etc. for each iteration

## Example Full Workflow

```bash
# Day 1: Start dev work
npm version 1.2.0-dev.1
npm run build
npm publish --tag dev --access public

# Team tests with: npx @insforge/install --client cursor --env API_KEY=key --dev

# Day 2: Fix bugs
npm version 1.2.0-dev.2
npm run build
npm publish --tag dev --access public

# Day 3: Ready for production
npm version 1.2.0  # Remove -dev suffix
npm run build
npm publish --access public
git push && git push --tags
```

## Notes for AI Assistants

- **ALWAYS ask** whether to publish to dev or production
- **NEVER publish to production** without explicit user confirmation
- **ALWAYS run build** before publishing
- **CHECK package.json** for current version before bumping
- When publishing dev: Use `--tag dev --access public`
- When publishing prod: Use `--access public` only (no --tag)

---
> Source: [InsForge/insforge-mcp](https://github.com/InsForge/insforge-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-07 -->
