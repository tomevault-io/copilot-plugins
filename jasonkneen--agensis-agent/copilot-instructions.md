## agensis-agent

> This repository is the open-source Agensis host agent. It does not contain the

# AGENTS.md

This repository is the open-source Agensis host agent. It does not contain the
Agensis website, backend, database, or desktop application.

## Layout

- `packages/agensis-cli/src` is the readable source of truth.
- `packages/agensis-agent` builds the published single-file npm bundle.
- `tests` contains Node and Vitest coverage for the daemon.

## Versioning

Keep these values identical for every release:

1. root `package.json` version
2. both package versions
3. `AGENSIS_CLI_VERSION` in `packages/agensis-cli/src/agensis.mjs`
4. `SOURCE_VERSION` in `packages/agensis-agent/build.mjs`
5. generated lockfile package versions

## Self-update + rollback (`agensis supervise`)

Optional: run `agensis supervise [--profile <name>]` instead of `agensis connect`
to let an agent trigger an update+reload of its own daemon connection, with
automatic rollback if the new version doesn't come up healthy.

- **Trigger**: the agent (or anything with --add-dir access to its own state
  dir) writes `~/.agensis/<workspace>/<agent>/update-request.json`
  (`{"targetVersion": "0.1.31"}`) — see `writeUpdateRequest` in
  `packages/agensis-cli/src/state.mjs`. No new network surface.
- **Install layout**: versions live side-by-side under
  `~/.agensis/versions/<version>/` (each an independent
  `npm install --prefix`), with `~/.agensis/versions/current` a symlink to the
  one actually running. Rollback is a symlink flip + relaunch, never a second
  install, so a broken new version can never block recovering the last-known-
  good one — see `packages/agensis-cli/src/selfUpdate.mjs`.
- **Health check**: "registered + heartbeating" (a fresh, connected
  `heartbeat.json` from the new process's pid), not "claimed a job" — an idle
  agent with no queued work would never pass a job-claim bar.
- **State**: `~/.agensis/<workspace>/<agent>/update.json` records
  `currentVersion`/`previousVersion`/`lastAttempt` (daemon/supervisor-owned;
  see `packages/agensis-cli/src/supervise.mjs`).
- **Surviving a killed supervisor**: `agensis supervise` is a separate process
  from the daemon it spawns, deliberately dumb and stable, so it keeps running
  across daemon crashes/updates. It should be run under a process manager with
  its own restart policy (systemd `Restart=always`, launchd `KeepAlive`, pm2)
  so that even if it is itself killed, the OS brings it back; on restart it
  re-derives everything from disk (no in-memory state to lose).
- Scoped to host daemons for now — `run_mode: 'sandbox'` (e2b) jobs run in an
  ephemeral VM with no persistent installed daemon to update in this sense.

## Verification

Run `npm run verify`. Do not publish unless it passes and the generated bundle
reports the intended version through `node packages/agensis-agent/bin/agensis.mjs --version`.

## Release

Push a matching `agent-v<version>` tag. The publish workflow verifies the tag,
builds and packs the bundle, and publishes `@agensis/agensis-agent` through the
npm trusted-publisher binding for `jasonkneen/agensis-agent` and
`.github/workflows/publish-agent.yml`. It uses GitHub OIDC and has no long-lived
npm token secret.

---
> Source: [jasonkneen/agensis-agent](https://github.com/jasonkneen/agensis-agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-27 -->
