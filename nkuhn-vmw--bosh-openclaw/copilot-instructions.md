## bosh-openclaw

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

BOSH release and Tanzu OpsMan 3.0 tile that deploys [OpenClaw](https://github.com/openclaw/openclaw) AI agents as a **multi-tenant CF service**. A Go service broker provisions a **dedicated BOSH VM per developer** on `cf create-service`, each with its own WebChat UI route, optional browser automation, and optional Mission Control integration. Implements OSB API v2.16 (async).

## Commands

- `./scripts/add-blob.sh` — fetch OpenClaw upstream blobs
- `./scripts/build-tile.sh` / `build-tile-dev.sh` — build `.pivotal`
- `bosh create-release --force && bosh upload-release`
- `bosh -d openclaw deploy manifests/openclaw.yml` (or `openclaw-lite.yml`)
- `om-cli-cdc.yml` / `om-cli-tdc.yml` — `om` config payloads for CDC and TDC foundations
- CI: `.github/workflows/{ci,openclaw-watcher}.yml` (watcher auto-bumps on upstream releases)

## Architecture / what to know

- **Static** deployment: `openclaw-broker` VM (Go, colocated route-registrar + BPM). **On-demand** deployments: one per `cf create-service`, each running `openclaw-agent` (+ gateway, webchat, route-registrar, optional headless browser)
- Broker drives the BOSH Director API directly to provision/update/delete agent deployments — not via a runtime addon
- Agent routes are dynamic: `oc-<owner>-<id>.<apps-domain>` registered via NATS TLS to gorouter
- `upgrade-agents` errand rolls all existing service instances to the current agent version (canary supported); `backup-agents` dumps per-instance state
- Real source lives under `src/`: `openclaw-broker`, `openclaw-gateway-custom`. The `openclaw` package pulls upstream binary blobs.
- LLM provider is pluggable: Tanzu GenAI Service (auto-discovered via CF marketplace service key), Anthropic, OpenAI, any OpenAI-compatible endpoint. Per-instance model override via `-c '{"model":"..."}'`.
- **Mission Control**: optional integration — broker registers each new agent with a Mission Control deployment when `mc_enabled=true`. Tile form under "Mission Control" tab.

## Gotchas

- OpsMan **3.0+** required; stemcell ubuntu-jammy 1.*
- Requires CF with gorouter **and NATS TLS** — non-TLS NATS foundations will fail route registration silently
- Runtime config edits from the Control UI are persisted via deep-merge on redeploy; a tile re-stage that drops the config file will lose them
- Broker needs BOSH Director creds with deployment-create rights — scoped UAA client, not admin, is preferred
- Packages pin specific Go (`golang-1.24`) and Node (`node22`) — don't bump casually, OpenClaw upstream tracks these

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/nkuhn-vmw) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
