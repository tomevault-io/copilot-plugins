## trash-guides-glossary

> TRaSH Guides — Glossary (arr stack, hardlinks, quality, indexers)


# TRaSH Guides — Glossary

> **Agent:** TRaSH Guides are the **gold standard** for terminology. Follow them as well as possible. **Open the links** in this rule (or on [trash-guides.info](https://trash-guides.info/)) to get the full definitions when writing or clarifying media-stack docs.

**Source:** [TRaSH Guides – Glossary](https://trash-guides.info/Glossary/)

Use these terms consistently when editing media stack docs and compose.

## Starr Apps (*arrs)

- **Sonarr** — TV automation. **Radarr** — Movies. **Lidarr** — Music. **Prowlarr** — Indexers/trackers. **Bazarr** — Subtitles.

## Paths and moves

- **Atomic move / Instant move** — Move in place (instant), not copy then delete. Required for Usenet with *arrs.
- **Hardlink** — Same file in multiple locations, one copy on disk; only within the same file system. Used for perma-seed + library. Modifying one copy modifies all.

## Quality and formats

- **Custom Format (CF)** — Pattern matching on release names at search time; scores in Quality Profiles. After import, with TRaSH naming, CF score can match the actual file properties.
- **Quality Profile (QP)** — Group and order of qualities, Custom Format scores, and related settings per movie/show/collection.
- **Quality** — Source + optional resolution (e.g. Bluray-2160p, Remux-1080p).
- **Quality Definitions** — Min/preferred/max file size per quality (MB per minute).

## Download and indexers

- **Indexer** — Source for NZBs (Usenet) or torrents; in *arr context “indexers” often means both Usenet indexers and torrent trackers.
- **Tracker** — Server that helps peers find each other for a torrent; also used for sites that host .torrent files.
- **Perma-seed** — Seed torrents long-term; often with hardlinks so library and torrent share one file.
- **Seeder** — Has full torrent and shares it. **Leecher** — Downloading. **Peer** — Seeder or leecher. **Swarm** — All peers for a torrent.

## Other

- **PAR (PAR2)** — Usenet parity/repair files.
- **USP** — Usenet Service Provider.
- **qBittorrent** — Avoid new major/minor versions until community-validated (often bugs).

---
> Source: [amazor/Self-Hosting](https://github.com/amazor/Self-Hosting) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-29 -->
