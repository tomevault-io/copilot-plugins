## dot-bin

> Manages portable CLI tool binaries with automated updates from upstream GitHub/GitLab/Kubernetes releases. Each tool is defined by a JSON file in `packages/`, and `scripts/update.sh` handles downloading, verifying, extracting, and installing binaries for both x86_64 and arm64 architectures. CI creates PRs for package updates; merging to master triggers a release via GitHub Actions.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

Manages portable CLI tool binaries with automated updates from upstream GitHub/GitLab/Kubernetes releases. Each tool is defined by a JSON file in `packages/`, and `scripts/update.sh` handles downloading, verifying, extracting, and installing binaries for both x86_64 and arm64 architectures. CI creates PRs for package updates; merging to master triggers a release via GitHub Actions.

## Quick Reference

```bash
# Update all packages to latest upstream versions (local dev)
./scripts/update.sh

# Update specific packages
./scripts/update.sh nvim lazygit gh

# Update to a custom output directory
DOT_BIN_DIR=/tmp/staging/bin ./scripts/update.sh

# Override release channel at runtime (stable or unstable)
CHANNEL=unstable ./scripts/update.sh nvim

# Verify a binary after install
file bin/x86_64/<name>    # should show "ELF 64-bit LSB ... x86-64"
file bin/arm64/<name>     # should show "ELF 64-bit LSB ... ARM aarch64"

# List release assets for a GitHub repo (useful when adding packages)
gh release view --repo <owner/repo> --json assets --jq '.assets[].name'

# Check current installed versions
cat versions.json
```

## Package JSON Format

Each package is defined in `packages/<name>.json`. Fields:

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `name` | string | yes | Package identifier, must match filename |
| `repo` | string | yes* | GitHub `owner/repo` (*omit for GitLab/Kubernetes sources) |
| `source` | string | no | `"github"` (default), `"gitlab"`, or `"kubernetes"` |
| `gitlab_project` | string | no | URL-encoded GitLab project path (required when `source` is `"gitlab"`) |
| `tag_prefix` | string | no | Prefix stripped from git tag to get version (e.g. `"v"`, `"cli-v"`, `"jq-"`, `""`) |
| `channel` | string | no | `"stable"` (default) or `"unstable"` -- stable uses latest non-pre-release; unstable includes pre-releases |
| `format` | string | yes | `"tarball"`, `"zip"`, or `"binary"` |
| `output_binaries` | string[] | yes | Binary names placed in `bin/{arch}/` |
| `checksum.asset` | string | no | Checksum filename in the release (supports `{version}` placeholder) |
| `checksum.algorithm` | string | no | `"sha256"` or `"sha512"` |
| `architectures` | object | yes | Per-arch download and extract config (keys: `x86_64`, `arm64`) |
| `architectures.{arch}.asset_pattern` | string | yes | Download filename (supports `{version}` placeholder) |
| `architectures.{arch}.extract_path` | string or string[] | no | Path inside archive; use array for multi-binary packages; supports wildcards (e.g. `*/bin/gh`) |
| `architectures.{arch}.checksum_asset` | string | no | Per-arch checksum file, overrides `checksum.asset` (for per-asset checksums) |

### Examples

Simple tarball with checksum (`lazygit`):
```json
{
  "name": "lazygit",
  "repo": "jesseduffield/lazygit",
  "tag_prefix": "v",
  "format": "tarball",
  "output_binaries": ["lazygit"],
  "checksum": { "asset": "checksums.txt", "algorithm": "sha256" },
  "architectures": {
    "x86_64": { "asset_pattern": "lazygit_{version}_linux_x86_64.tar.gz", "extract_path": "lazygit" },
    "arm64": { "asset_pattern": "lazygit_{version}_linux_arm64.tar.gz", "extract_path": "lazygit" }
  }
}
```

Standalone binary, no archive (`jq`):
```json
{
  "name": "jq",
  "repo": "jqlang/jq",
  "tag_prefix": "jq-",
  "format": "binary",
  "output_binaries": ["jq"],
  "checksum": { "asset": "sha256sum.txt", "algorithm": "sha256" },
  "architectures": {
    "x86_64": { "asset_pattern": "jq-linux-amd64" },
    "arm64": { "asset_pattern": "jq-linux-arm64" }
  }
}
```

Multi-binary with per-asset checksums (`uv`):
```json
{
  "name": "uv",
  "repo": "astral-sh/uv",
  "tag_prefix": "",
  "format": "tarball",
  "output_binaries": ["uv", "uvx"],
  "checksum": { "algorithm": "sha256" },
  "architectures": {
    "x86_64": {
      "asset_pattern": "uv-x86_64-unknown-linux-gnu.tar.gz",
      "extract_path": ["uv-x86_64-unknown-linux-gnu/uv", "uv-x86_64-unknown-linux-gnu/uvx"],
      "checksum_asset": "uv-x86_64-unknown-linux-gnu.tar.gz.sha256"
    },
    "arm64": {
      "asset_pattern": "uv-aarch64-unknown-linux-gnu.tar.gz",
      "extract_path": ["uv-aarch64-unknown-linux-gnu/uv", "uv-aarch64-unknown-linux-gnu/uvx"],
      "checksum_asset": "uv-aarch64-unknown-linux-gnu.tar.gz.sha256"
    }
  }
}
```

GitLab source (`glab`):
```json
{
  "name": "glab",
  "source": "gitlab",
  "gitlab_project": "gitlab-org%2Fcli",
  "tag_prefix": "v",
  "format": "tarball",
  "output_binaries": ["glab"],
  "checksum": { "asset": "checksums.txt", "algorithm": "sha256" },
  "architectures": {
    "x86_64": { "asset_pattern": "glab_{version}_linux_amd64.tar.gz", "extract_path": "bin/glab" },
    "arm64": { "asset_pattern": "glab_{version}_linux_arm64.tar.gz", "extract_path": "bin/glab" }
  }
}
```

Kubernetes source (`kubectl`):
```json
{
  "name": "kubectl",
  "source": "kubernetes",
  "tag_prefix": "v",
  "format": "binary",
  "output_binaries": ["kubectl"],
  "checksum": { "algorithm": "sha256" },
  "architectures": {
    "x86_64": { "asset_pattern": "kubectl", "checksum_asset": "kubectl.sha256" },
    "arm64": { "asset_pattern": "kubectl", "checksum_asset": "kubectl.sha256" }
  }
}
```

## Adding a New Package

1. Find the repo and examine its latest release assets
2. Identify Linux x86_64 and arm64 asset filenames
3. Determine format (`tarball`/`zip`/`binary`), tag prefix, and whether checksums exist
4. For archives, download one and list contents to find the binary path inside
5. Create `packages/<name>.json` with all required fields
6. Run `./scripts/update.sh <name>` to test
7. Verify binaries: `file bin/x86_64/<name>` and `file bin/arm64/<name>`
8. Commit: `git add packages/<name>.json versions.json`

## Architecture

### Scripts

**`scripts/lib.sh`** -- shared functions sourced by update.sh:
- `BIN_DIR="${DOT_BIN_DIR:-${REPO_ROOT}/bin}"` -- output directory, overridable via env var
- `ARCHITECTURES=("x86_64" "arm64")` -- supported architectures
- `get_current_version(name)` -- reads version from `versions.json`
- `set_version(name, version)` -- writes version to `versions.json`
- `strip_prefix(tag, prefix)` -- removes tag prefix to get clean version
- `get_latest_tag(pkg_file)` -- queries GitHub API (`gh api`), GitLab API, or Kubernetes stable endpoint for latest release tag; respects `channel` setting (`stable`/`unstable`) and `CHANNEL` env var override
- `download_asset(pkg_file, tag, dest_dir, arch)` -- downloads the per-arch release asset via `gh release download` (GitHub), `curl` (GitLab), or Kubernetes download URL
- `download_checksum_file(pkg_file, tag, dest_dir, arch)` -- downloads checksum file, respects arch-specific overrides
- `verify_checksum(pkg_file, asset_name, asset_path, tag, arch)` -- validates SHA256/512 against downloaded checksum file
- `resolve_path(base_dir, pattern)` -- resolves extract paths including wildcard/glob patterns
- `install_binary(pkg_file, tmpdir, asset_name, arch)` -- extracts (tar/zip) or copies (binary) to `bin/{arch}/`
- `install_extracted(pkg_file, tmpdir, arch)` -- handles single and array extract_path, supports wildcards
- `download_and_install(pkg_file, tag)` -- orchestrates the full per-arch loop: download, verify, install, update version

**`scripts/update.sh`** -- main driver:
- Accepts optional package names as arguments (defaults to all)
- For each package: fetches latest tag, compares to current version, downloads and installs if newer
- Exits with code 1 if any package failed

**`scripts/bump-changelog.py`** -- automated CHANGELOG management:
- Usage: `bump-changelog.py <old-versions.json> <new-versions.json>`
- Diffs old/new versions to find added, updated, and removed packages
- Bumps patch version from latest CHANGELOG entry (e.g. `0.4.0` -> `0.4.1`)
- Inserts a new changelog section with categorized changes (Updated/Added/Removed)
- Prints the new version to stdout; exits with code 2 if no changes detected

**`scripts/release-notes.py`** -- generates markdown release notes by diffing old/new `versions.json`

**`install.sh`** -- user-facing install/update script:
- Detects architecture (`uname -m`)
- Downloads latest release tarball via GitHub API
- Verifies SHA256 checksum
- Extracts to `~/.local/bin/` (or `$DOT_BIN_INSTALL_DIR`)

### Update Flow

```
update.sh [pkg...]
  for each package:
    get_latest_tag()          -- fetch latest release tag from API
    compare with versions.json
    if newer:
      for each arch (x86_64, arm64):
        download_asset()      -- download release asset to tmpdir
        verify_checksum()     -- validate if checksum configured
        install_binary()      -- extract/copy to bin/{arch}/
      set_version()           -- update versions.json
```

### CI

Two GitHub Actions workflows handle the update-and-release pipeline:

**`update.yml`** -- daily package updates (06:00 UTC, or manual dispatch):
1. Checks out repo, saves current `versions.json`
2. Runs `update.sh` (all packages or filtered via input)
3. If `versions.json` changed, runs `bump-changelog.py` to bump version and update CHANGELOG.md
4. Creates/updates a PR (`chore/update-packages` branch) targeting master with the version and changelog changes

**`release.yml`** -- triggered on push to master:
1. Extracts the latest version from CHANGELOG.md (first `## [x.y.z]` heading)
2. Skips if no version found or release already exists for that tag
3. Downloads all binaries by running `update.sh` with empty `versions.json`
4. Creates `dot-bin-{arch}.tar.gz` tarballs with SHA256 checksums
5. Extracts release notes from CHANGELOG.md for the current version
6. Publishes a GitHub Release tagged `v{version}` with tarballs, checksums, and versions.json

**`ci.yml`** -- PR validation:
1. Detects changed package JSON files in the PR
2. Downloads and installs changed packages (or all if no package changes)
3. Verifies all binaries are valid ELF 64-bit executables
4. Validates `versions.json` is valid JSON

### Release Distribution

- **Tagging**: Semantic version-based (`v0.4.0`), driven by CHANGELOG.md version
- **Tarball structure**: Flat binaries at top level -- `tar xzf dot-bin-x86_64.tar.gz -C ~/.local/bin/` just works
- **Install**: `curl -fsSL .../install.sh | bash` downloads latest release to `~/.local/bin/`

## Conventions

- **Bash style**: `set -euo pipefail` at top of every script; pass shellcheck
- **JSON**: 2-space indent, one package per file in `packages/`, filename matches `name` field
- **bin/**: Gitignored, not tracked in the repo. Binaries are distributed via GitHub Releases
- **versions.json**: Auto-generated by update.sh, do not edit manually
- **CHANGELOG.md**: Auto-updated by `bump-changelog.py` during CI; manual edits for feature releases
- **Commit messages**: Conventional commits (`feat:` for new packages, `chore:` for updates, `fix:` for bug fixes)

---
> Source: [Honzys/dot-bin](https://github.com/Honzys/dot-bin) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-28 -->
