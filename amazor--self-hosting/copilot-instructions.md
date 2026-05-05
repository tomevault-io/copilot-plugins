## trash-guides-radarr

> TRaSH Guides — Radarr configuration reference


# TRaSH Guides — Radarr

> **Agent:** TRaSH Guides are the **gold standard** for Radarr. Follow them as well as possible. **Open the links** in this rule (or on [trash-guides.info](https://trash-guides.info/)) to get the correct, up-to-date guide when configuring or documenting Radarr.

**Source:** [TRaSH Guides – Radarr](https://trash-guides.info/Radarr/)

## Configuration order (after paths)

1. **Quality Settings (File Size)** — Avoid low-quality/fake releases.
2. **Recommended naming scheme** — Prevents download loops (see below).
3. **Quality Profiles** — Custom Formats, ordering. Variants: [Anime](https://trash-guides.info/Radarr/Radarr-quality-profile-setup-anime/), [French](https://trash-guides.info/Radarr/Radarr-setup-quality-profile-french/), [German](https://trash-guides.info/Radarr/Radarr-setup-quality-profile-german/).
4. **Custom Formats** — Import/update from guide; [Collection of Custom Formats](https://trash-guides.info/Radarr/Radarr-collection-of-custom-formats/).

## Naming (why include “extra” info)

- **Prevents duplicate downloads** and re-import issues. Media servers (Plex/Emby/Jellyfin) only need playable names; Radarr needs quality/source to avoid loops.
- **Non-recoverable information** (must be in the filename): Repack/Proper, Edition (Director’s Cut, Theatrical, etc.), Release group, Quality source (HDTV, WEB-DL, Blu-ray, Remux).
- In Radarr: enable **Show Advanced**, **Rename Movies**, and **Analyze video files** (Settings → Media Management).

## Standard movie format (Plex, TRaSH)

- `{Movie CleanTitle} {(Release Year)} - {{Edition Tags}} {[MediaInfo 3D]}{[Custom Formats]}{[Quality Full]}{[Mediainfo AudioCodec}{ Mediainfo AudioChannels]}{[MediaInfo VideoDynamicRangeType]}{[Mediainfo VideoCodec]}{-Release Group}`
- Example: `The Movie Title (2010) {edition-Ultimate Extended Edition} [Bluray-1080p Proper][DV HDR10][DTS 5.1][x264]-RlsGrp`
- For Plex edition in UI use `{edition-{Edition Tags}}`; for one entry per movie when keeping multiple copies use `{Edition Tags}` only.
- Folder: `{Movie CleanTitle} ({Release Year})` minimum; optional `{tmdb-{TmdbId}}` or `{imdb-{ImdbId}}` for Plex/Emby/Jellyfin.

Full schemes (Plex/Emby/Jellyfin, IMDb/TMDb): [Recommended naming scheme](https://trash-guides.info/Radarr/Radarr-recommended-naming-scheme/).

## Tips

- **Remote Path Mappings** — When Radarr and download client see different paths (e.g. Docker).
- Merge quality, order quality sources, rename root folders, language Custom Formats: [Radarr Tips](https://trash-guides.info/Radarr/).

---
> Source: [amazor/Self-Hosting](https://github.com/amazor/Self-Hosting) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-29 -->
