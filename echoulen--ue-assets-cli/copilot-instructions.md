## ue-assets-cli

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Is

`ue-assets` is a Node.js CLI tool (published to npm as `ue-assets`) that installs and updates Unreal Engine plugins/content from GitHub Releases. It reads a JSON config, compares against a lock file, downloads ZIP releases via the GitHub API, and extracts them into a target directory.

Requires `GITHUB_TOKEN` in `.env` (or environment) with `contents:read` scope.

## Commands

```bash
# Run locally during development
node cli.js install --config plugins.json --dir ./Plugins
node cli.js update  --config plugins.json --dir ./Plugins

# Install dependencies
npm install
```

No build step, no tests, no linter configured.

## Architecture

```
cli.js              # Commander CLI entry point; loads .env, routes to handlers
lib/
  install.js        # installOne() + installAll(); reads config, checks lockfile, downloads+extracts
  update.js         # fetchLatest() + updateAll(); updates config JSON in-place, calls installOne(clean=true)
  github.js         # Lazy Octokit init; getLatestTag(), listAssetNames(), downloadAsset()
  extract.js        # ZIP extraction; auto-detects 3 structural patterns (Plugins/name/*, name/*, flat)
  lockfile.js       # lockPathFor(), getLocked(), writeLock() — derives lock path from config path
```

**Data flow for `install`:**
1. Read config JSON (root key can be `"plugins"` or `"content"`)
2. For each entry, check lock file — skip if version already installed
3. Download asset ZIP to `/tmp/ue-assets-{pid}/`
4. Extract to `--dir` using smart structure detection
5. Write lock file entry

**Data flow for `update`:**
1. Call `getLatestTag()` per repo to find newest release
2. Verify the expected asset name exists in that release
3. Patch version in config JSON in-place (preserves formatting via string replace)
4. Call `installOne()` with `clean=true` to force re-extraction
5. Rollback config on any failure

## Config Format

```json
{
  "dir": "../../Plugins",
  "plugins": {
    "MyPlugin": {
      "repo": "owner/repo",
      "version": "v1.2.3",
      "asset": "MyPlugin-{version}.zip"
    }
  }
}
```

- `dir` (optional): default install directory; CLI `--dir` overrides it; falls back to `Plugins`
- `--config` defaults to `plugins` and auto-appends `.json` if no extension given
- The `{version}` placeholder in `asset` is substituted at runtime. Lock file is auto-derived: `plugins.json` → `plugins-lock.json`.

## Publishing

Releases publish automatically via `.github/workflows/publish.yml` when a GitHub Release is created. The workflow sets the npm version from the git tag before publishing.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/echoulen) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
