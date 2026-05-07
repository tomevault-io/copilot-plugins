## codex-whatsapp-relay

> Use this repo's release process every time a new version ships.

# Agent Notes

## Release Workflow

Use this repo's release process every time a new version ships.

1. Work from `main`, not `master`.
2. Make sure the target changes are already merged into `main`.
3. Update `CHANGELOG.md` so `## [Unreleased]` contains the release notes you want to ship.
4. Run the deterministic release prep script with explicit inputs:
5. `npm run release:prepare -- --version X.Y.Z --date YYYY-MM-DD`
6. The script is the source of truth for release-managed file updates. It updates all of these together:
7. `CHANGELOG.md`
8. `package.json`
9. `package-lock.json`
10. `plugins/whatsapp-relay/.codex-plugin/plugin.json`
11. `README.md` release-pinned install tag references
12. Use `--dry-run` first if you want a no-write preview.
13. Review the diff after the script runs.
14. Run `npm run check`.
15. Run `npm test`.
16. If the release touches voice replies or local TTS routing, run a smoke test with `npm run whatsapp:tts:smoke`.
17. Commit the release preparation changes on `main`.
18. Push `main`.
19. Create and push an annotated git tag in the format `vX.Y.Z`.
20. Publish the GitHub release using the matching changelog entry as the release notes.

## Local Plugin Reinstall

When refreshing the locally installed Codex plugin after a release:

1. Reinstall from the release tag, not from a floating branch.
2. The installed plugin repo lives at `~/.codex/plugins/whatsapp-relay`.
3. Fetch tags, check out the desired release tag, and run `npm install`.
4. Do not force a new WhatsApp authentication flow if existing auth state is available.
5. If needed, copy local runtime state from the main repo plugin data into the installed plugin data:
6. `plugins/whatsapp-relay/data/auth`
7. `plugins/whatsapp-relay/data/store.json`
8. `plugins/whatsapp-relay/data/controller-config.json`
9. Do not copy `controller-state.json` across installs because it can preserve stale process IDs.

## Notes

- The repo's primary branch is `main`.
- The canonical repository URL is `https://github.com/abuiles/codex-whatsapp-relay`. Keep plugin metadata and README install instructions aligned with the actual git remote.
- The README install prompt is part of the release surface and must be updated on every new release.
- The goal is that a fresh Codex install follows the latest tagged release by default.

---
> Source: [abuiles/codex-whatsapp-relay](https://github.com/abuiles/codex-whatsapp-relay) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-03 -->
