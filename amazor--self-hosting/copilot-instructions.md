## trash-guides-plex

> TRaSH Guides — Plex server and client settings


# TRaSH Guides — Plex

> **Agent:** TRaSH Guides are the **gold standard** for Plex server and client settings. Follow them as well as possible. **Open the links** in this rule (or on [trash-guides.info](https://trash-guides.info/)) to get the correct, up-to-date guide when configuring or documenting Plex (including accelerated/transcoding).

**Source:** [TRaSH Guides – Plex](https://trash-guides.info/Plex/)

## Server

- **Suggested Plex Media Server Settings** — [Suggested settings](https://trash-guides.info/Plex/Plex-media-server-settings/) for recommended server behavior.
- **Stop 4K transcoding** — [Stop 4K Video Transcoding](https://trash-guides.info/Plex/Stop-4k-transcoding/) — Use Tautulli + JBOPS (Plex Pass required) to block 4K transcodes and preserve quality/bandwidth.

## Client and playback

- **Optimal Plex Client Settings** — [Optimal client settings](https://trash-guides.info/Plex/Optimal-Plex-Client-Settings/) (3rd party) per client.
- **What does my media player support** — [Media player support](https://trash-guides.info/Plex/What-does-my-media-player-support/) — Community-tested devices.
- **Profiles** — Transcoding/codec behavior.

When editing `docker_compose/accelerated/` or Plex in `docker_compose/media/`: use the same path root (e.g. `/data/media`) as in File and Folder Structure; give Plex access only to the media library, not the download folder.

---
> Source: [amazor/Self-Hosting](https://github.com/amazor/Self-Hosting) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-29 -->
