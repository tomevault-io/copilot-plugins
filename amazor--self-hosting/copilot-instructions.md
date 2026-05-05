## trash-guides-file-folder-structure

> TRaSH Guides — File and folder structure for media (hardlinks, paths)


# TRaSH Guides — File and Folder Structure

> **Agent:** TRaSH Guides are the **gold standard** for this topic. Follow them as well as possible. **Open the links** in this rule (or on [trash-guides.info](https://trash-guides.info/)) to get the correct, up-to-date guide when configuring or documenting file/folder structure, paths, or hardlinks.

**Source:** [TRaSH Guides – File and Folder Structure](https://trash-guides.info/File-and-Folder-Structure/)

## Requirements

- All media and downloads must be on the **same file system** (one physical or virtual drive).
- All applications must have a **consistent view** of paths — files appear in the same place for every app.

## Recommended layout (host)

```
data
├── torrents
│   ├── books
│   ├── movies
│   ├── music
│   └── tv
├── usenet
│   ├── incomplete
│   └── complete
│       ├── books
│       ├── movies
│       ├── music
│       └── tv
└── media
    ├── books
    ├── movies
    ├── music
    └── tv
```

The `data` folder can live anywhere (e.g. Unraid share `data` → `/mnt/user/data`). **Pick one root** (`/data`, `/shared`, `/storage`) and use it for every app.

## Docker

- Grant each container access to the **lowest-level folder it needs** while keeping the **same top-level path** inside the container.
- Example: host `/mnt/user/data/torrents` → container `/data/torrents`. So the download client sees `/data/torrents`; Sonarr/Radarr see `/data/media`, `/data/torrents`, etc. Same `/data` prefix everywhere.

## Hardlinks

- Same file can appear in multiple places without extra disk space. Used for perma-seed + library.
- **Limitations:** Cannot hardlink directories. Cannot hardlink across separate file systems, partitions, volumes, or mounts. Avoid exFAT.
- Modifying any copy of a hardlinked file changes all copies (e.g. changing ID3 on one copy affects the other — avoid after import for torrents).

## Instant moves (atomic moves)

- A real move, not copy-then-delete. Required for fast Usenet imports.

## App menu reference (TRaSH Examples)

- **Sonarr:** Settings → Media Management → Importing / Root Folders.
- **Radarr:** Settings → Media Management → Importing / Root Folders.
- **SABnzbd:** Config → Folders / Categories.
- **qBittorrent:** Options → Downloads.

More: [How to set up](https://trash-guides.info/File-and-Folder-Structure/How-to-set-up/) (Docker, Synology, Native, etc.).

---
> Source: [amazor/Self-Hosting](https://github.com/amazor/Self-Hosting) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-29 -->
