## trash-guides-prowlarr

> TRaSH Guides — Prowlarr (indexers) reference


# TRaSH Guides — Prowlarr

> **Agent:** TRaSH Guides are the **gold standard** for Prowlarr. Follow them as well as possible. **Open the links** in this rule (or on [trash-guides.info](https://trash-guides.info/)) to get the correct, up-to-date guide when configuring or documenting indexers or Prowlarr.

**Source:** [TRaSH Guides – Prowlarr](https://trash-guides.info/Prowlarr/)

Centralized indexer management for Radarr, Sonarr, Lidarr, etc.

## Key guides (follow in TRaSH)

- **Proxy for certain indexers** — [How to set up proxy](https://trash-guides.info/Prowlarr/How-to-set-up-proxy-for-certain-indexers/) — Use VPN/proxy only for indexers that need it, not app-wide.
- **FlareSolverr** — [Setup FlareSolverr](https://trash-guides.info/Prowlarr/How-to-set-up-flaresolverr/) for indexers behind Cloudflare or DDoS-GUARD.
- **Indexers with limited API** — [Limited API setup](https://trash-guides.info/Prowlarr/How-to-set-up-indexers-with-limited-api/) — Avoid hitting API limits when using unpaid indexers as backup (trick to spread usage).

When adding Prowlarr to `docker_compose/media/`: add FlareSolverr service if any indexers require it; document proxy only for indexers that need it.

---
> Source: [amazor/Self-Hosting](https://github.com/amazor/Self-Hosting) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-29 -->
