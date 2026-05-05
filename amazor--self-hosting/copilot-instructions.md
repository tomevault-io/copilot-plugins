## trash-guides-guide-sync

> TRaSH Guides — Guide Sync tools (Recyclarr, Notifiarr, Configarr)


# TRaSH Guides — Guide Sync

> **Agent:** TRaSH Guides are the **gold standard** for syncing guide settings to *arrs. Follow them as well as possible. **Open the links** in this rule (or on [trash-guides.info](https://trash-guides.info/)) to get the correct, up-to-date guide when configuring or documenting Recyclarr, Notifiarr, or Configarr.

**Source:** [TRaSH Guides – Guide Sync](https://trash-guides.info/Guide-Sync/)

Third-party tools that sync TRaSH Guide settings (Custom Formats, Quality Profiles, naming, quality definitions) to Radarr and Sonarr. Use when configuring **configuration discipline** (e.g. Recyclarr) in the media VM.

## Options

- **Recyclarr** — CLI; config files to sync Custom Formats, scores, Quality Settings, Naming, Quality Profiles. No GUI. [Recyclarr](https://recyclarr.dev/).
- **Notifiarr** — GUI, notifications, and TRaSH sync (Patron feature); predefined profiles, flowchart for Custom Formats. [Notifiarr](https://notifiarr.com/).
- **Configarr** — For containerized/cron (Docker, Kubernetes); compatible with Recyclarr templates (pre v7.4.0); Custom Formats, Quality Profiles, Naming. [Configarr](https://github.com/Configarr/Configarr).

When documenting or adding Recyclarr/Configarr in `docker_compose/media/`, point to [Guide Sync](https://trash-guides.info/Guide-Sync/) and the tool’s docs for syncing Custom Formats and profiles.

---
> Source: [amazor/Self-Hosting](https://github.com/amazor/Self-Hosting) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-29 -->
