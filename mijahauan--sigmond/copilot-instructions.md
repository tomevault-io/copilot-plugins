## sigmond

> **Sigmond** ("Dr. SigMonD" — a play on Sigmund Freud) is the unified

# CLAUDE.md — Sigmond Development Briefing

## What this project is

**Sigmond** ("Dr. SigMonD" — a play on Sigmund Freud) is the unified
installer, configurator, and lifecycle manager for the HamSCI SDR suite:
wsprdaemon, hf-timestd, ka9q-radio, ka9q-web, and future HamSCI clients.

The CLI entry point is **`smd`**.

## Authors

- Rob Robinett (AI6VN) — wsprdaemon architect
- Michael Hauan (AC0G, GitHub: mijahauan) — hf-timestd / ka9q-python author
- Repo: https://github.com/mijahauan/sigmond

## Architecture reference

See `tui-configurator.md` for the full design — three-panel Textual TUI,
topology registry, harmonization rules, screen flow, and open questions.

## Core commands (target CLI)

```
smd install    Install & configure all components
smd apply      Reconcile running services with current config (like wd-ctl apply)
smd start      Start all managed services
smd stop       Stop all managed services
smd status     Show service health (all components)
smd config     Edit configuration (TUI)
smd log        View service logs
smd update     Update dependencies to latest pinned commits, rebuild if needed
smd diag       Run diagnostics and cross-component validation
```

## Implementation priorities

1. **`smd install`** — first priority. Installs all components from scratch:
   - Clones/updates dependency repos (from a deps manifest)
   - Builds repos that require compilation (ka9q-radio, ka9q-web, onion)
   - Installs Python packages into the appropriate venvs
   - Copies `smd` itself to `/usr/local/sbin/`
   - Installs systemd units
   - Generates env files from config
   - Enables and starts services

2. **`smd apply`** — reconciles running services with current config without
   a full reinstall. Diff desired vs actual, restart only what changed.

3. **`smd start` / `smd stop`** — simple wrappers around `systemctl
   start/stop` for the full service set.

4. **`smd status`** — aggregated health across all managed services.

5. **TUI configurator** — built on top of the above once the headless commands
   are solid. Uses Textual (Python). See `tui-configurator.md`.

## Topology registry

```toml
# /etc/sigmond/topology.toml
[radiod]
host = "localhost"
managed = true
config = "/etc/ka9q-radio/..."

[wspr-recorder]
config = "/etc/wsprdaemon/wsprdaemon.conf"
managed = true

[hf-timestd]
config = "/etc/hf-timestd/timestd-config.toml"
managed = true
```

## Key constraints

- **Primary language:** Python 3.11, stdlib only for the core (`smd`).
  Textual is a runtime dep for the TUI subcommand only.
- **No wdlib dependency.** Sigmond is a separate tool; it may read
  wsprdaemon config but does not import wdlib.
- **Headless-first.** Every command must work without a terminal (for
  remote SSH, CI, and scripted installs). TUI is additive.
- **FHS-compliant paths:**
  - Config: `/etc/sigmond/`
  - Binaries: `/usr/local/sbin/smd`
  - Logs: `/var/log/sigmond/`
  - State: `/var/lib/sigmond/`

## Companion project

`wsprdaemon-client` lives at `/home/wsprdaemon/wsprdaemon-client` and is
the repo Sigmond will install and manage. Its `deps.conf` is the
authoritative source for dependency commit pins.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/mijahauan) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
