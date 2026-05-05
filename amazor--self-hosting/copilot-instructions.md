## trash-guides-sabnzbd

> TRaSH Guides — SABnzbd paths and categories


# TRaSH Guides — SABnzbd

> **Agent:** TRaSH Guides are the **gold standard** for SABnzbd in the media stack. Follow them as well as possible. **Open the links** in this rule (or on [trash-guides.info](https://trash-guides.info/)) to get the correct, up-to-date guide when configuring or documenting SABnzbd paths or categories.

**Source:** [TRaSH Guides – SABnzbd](https://trash-guides.info/Downloaders/SABnzbd/)

Paths must support [hardlinks and instant moves](https://trash-guides.info/File-and-Folder-Structure/Hardlinks-and-Instant-Moves/). **Pick one path layout for all apps** (e.g. `/data`).

## Paths and categories

- **SABnzbd config → Folders** — Set **Completed Download Folder** (e.g. `/data/usenet` or `/data/usenet/complete`). **Config → Categories** — Add categories (e.g. `movies`, `tv`); folder/path for each category is **relative** to the “Relative folders are based on:” path (the Completed Download Folder). You do not need to enter the full path for each category.
- **Critical (TRaSH):**
  1. Plex/Emby/Jellyfin/Kodi should **only** have access to the **media** folder/library.
  2. Starr apps import (copy/move/hardlink) **from** the download location **to** the media folder.
  3. You tell Radarr/Sonarr where the clean media library is.
  4. The download client **only** downloads to the download folder.
  5. **Download and media library must never be the same location.**

## Other

- [Basic-Setup](https://trash-guides.info/Downloaders/SABnzbd/Basic-Setup/), [Paths and Categories](https://trash-guides.info/Downloaders/SABnzbd/Paths-and-Categories/), [Scripts](https://trash-guides.info/Downloaders/SABnzbd/Scripts/) (only if needed).

---
> Source: [amazor/Self-Hosting](https://github.com/amazor/Self-Hosting) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-29 -->
