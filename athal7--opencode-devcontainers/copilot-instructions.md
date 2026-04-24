## opencode-devcontainers

> Before committing changes, verify documentation is updated to reflect code changes:

# Agent Instructions

## Pre-Commit: Documentation Check

Before committing changes, verify documentation is updated to reflect code changes:

1. **README.md** - Update if changes affect:
   - Plugin usage (`/devcontainer`, `/worktree`, `/workspaces` commands)
   - Configuration options (`~/.config/opencode/devcontainers/config.json`)
   - Installation steps or dependencies
   - Usage examples

2. **CONTRIBUTING.md** - Update if changes affect:
   - Development setup or workflow
   - Test commands or patterns
   - Release process
   - Project structure

3. **plugin/command/*.md** - Update if changes affect:
   - Command arguments or behavior
   - Examples

## Post-PR: Release and Upgrade Workflow

After a PR is merged to main, semantic-release automatically:
1. Creates a new version based on commit messages
2. Publishes to npm
3. Creates a GitHub release

### Verify Release

```bash
gh release list -R athal7/opencode-devcontainers -L 1
npm view opencode-devcontainers version
```

### Upgrade

OpenCode automatically updates npm plugins on startup. Just restart OpenCode.

To force a specific version:
```json
{
  "plugin": ["opencode-devcontainers@1.0.0"]
}
```

### Config Locations

- Main config: `~/.config/opencode/devcontainers/config.json`
- Cache/state: `~/.cache/opencode-devcontainers/`
- Clones: `~/.local/share/opencode/clone/`
- Worktrees: `~/.local/share/opencode/worktree/`

If config format changed, check release notes:

```bash
gh release view -R athal7/opencode-devcontainers
```

---
> Source: [athal7/opencode-devcontainers](https://github.com/athal7/opencode-devcontainers) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-21 -->
