## freenet-agent-skills

> When making changes to the plugin:

# Freenet Agent Skills

## Updating the Version

When making changes to the plugin:

1. **Update marketplace.json**: Increment the version in `.claude-plugin/marketplace.json` → `metadata.version`
2. **Update CHANGELOG.md**: Add entry at the top with date and changes
3. **Commit both files**: Ensure both files are committed together

CI will fail if skills/agents/hooks are changed without a version bump in `marketplace.json`.

### Version Numbering

Follow semantic versioning (MAJOR.MINOR.PATCH):
- **MAJOR**: Breaking changes to skill structure or API
- **MINOR**: New skills or significant feature additions
- **PATCH**: Bug fixes, documentation updates, minor improvements

See [CHANGELOG.md](CHANGELOG.md) for full version history.

---
> Source: [freenet/freenet-agent-skills](https://github.com/freenet/freenet-agent-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-24 -->
