## trash-guides-qbittorrent

> TRaSH Guides — qBittorrent paths and categories


# TRaSH Guides — qBittorrent

> **Agent:** TRaSH Guides are the **gold standard** for qBittorrent in the media stack. Follow them as well as possible. **Open the links** in this rule (or on [trash-guides.info](https://trash-guides.info/)) to get the correct, up-to-date guide when configuring or documenting qBittorrent paths or categories.

**Source:** [TRaSH Guides – qBittorrent](https://trash-guides.info/Downloaders/qBittorrent/)

Paths must support [hardlinks and instant moves](https://trash-guides.info/File-and-Folder-Structure/Hardlinks-and-Instant-Moves/). **Pick one path layout for all apps** (e.g. `/data`); use the same root in qBittorrent and every *arr.

## Paths

- **Options → Downloads** — Set default save path (e.g. `/data/torrents`). Same path as in File and Folder Structure so completed downloads and media share one filesystem for hardlinks.

## Categories

- Categories let Starr apps (Sonarr, Radarr) track downloads by category instead of watching every torrent.
- **Add category:** Left panel → right-click or “Add category”. Set **Save path** to a subfolder name only (e.g. `tv`, `movies`) — it is relative to the default save path. Set **Category** to the same or a recognizable name (e.g. `sonarr`, `radarr`). You do **not** enter the full path under Save path; it uses **Options → Saving Management → Default Save Path**.
- **Critical:** Set **Default Torrent Management Mode** to **Automatic**. Otherwise downloads can end up in `/data/torrents/` instead of `/data/torrents/tv` or `/data/torrents/movies`, breaking category-based paths and *arr tracking.

## Other

- **Port forwarding** — [Port forwarding](https://trash-guides.info/Downloaders/qBittorrent/Port-forwarding/) for VPN.
- **Basic-Setup** — [Basic-Setup](https://trash-guides.info/Downloaders/qBittorrent/Basic-Setup/) for common settings.

---
> Source: [amazor/Self-Hosting](https://github.com/amazor/Self-Hosting) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-29 -->
