## unifi-protect-bridge-cli

> - `docs/live-testing.md`

# Agents

## Start Here

- `README.md`
- `.env.example`
- `docs/live-testing.md`
- `src/unifi_protect_bridge_cli/cli.py`
- `src/unifi_protect_bridge_cli/bridge_core/protect_api.py`
- `src/unifi_protect_bridge_cli/bridge_core/automation_plan.py`

## Notes

- This repository is the optional CLI. The Home Assistant/HACS integration lives in `Hovborg/unifi-protect-bridge`.
- The CLI is not required for HACS installs.
- Keep command behavior aligned with the Home Assistant integration domain `unifi_protect_bridge`.
- Do not print, commit, or persist real credentials. Use `.env.example` for placeholders only.
- Prefer environment variables over password flags in examples, so passwords do not land in shell history.
- `protect login-check`, `protect cameras`, `protect automations`, and `bridge diff` are read-only.
- `bridge apply --connect --yes` intentionally changes UniFi Protect, but must only touch bridge-owned automations with the `UniFi Protect Bridge:` prefix or legacy `HA Protect Bridge:` prefix.
- Prefer GitHub CLI (`gh`) for GitHub-specific work when it is available and authenticated.
- Use `gh run`, `gh workflow`, `gh pr`, and `gh api` for CI runs, workflow logs, PR state, and GitHub metadata.
- Keep using plain `git` for normal local repository operations such as diff, commit, and push.

---
> Source: [Hovborg/unifi-protect-bridge-cli](https://github.com/Hovborg/unifi-protect-bridge-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-21 -->
