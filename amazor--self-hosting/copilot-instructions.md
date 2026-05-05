## trash-guides-sonarr

> TRaSH Guides — Sonarr configuration reference


# TRaSH Guides — Sonarr

> **Agent:** TRaSH Guides are the **gold standard** for Sonarr. Follow them as well as possible. **Open the links** in this rule (or on [trash-guides.info](https://trash-guides.info/)) to get the correct, up-to-date guide when configuring or documenting Sonarr.

**Source:** [TRaSH Guides – Sonarr](https://trash-guides.info/Sonarr/)

## Configuration order (after paths)

1. **Quality Settings (File Size)** — Avoid low-quality or fake releases.
2. **Recommended naming scheme** — Prevents download loops; include non-recoverable info (repack/proper, edition, release group, quality source). Enable **Show Advanced** and **Rename Episodes**; **Analyze video files** in Media Management.
3. **Quality Profiles** — Custom Formats and ordering. Variants: [Anime](https://trash-guides.info/Sonarr/Sonarr-quality-profile-setup-anime/), [French](https://trash-guides.info/Sonarr/Sonarr-setup-quality-profile-french/), [German](https://trash-guides.info/Sonarr/Sonarr-setup-quality-profile-german/).
4. **Custom Formats** — Import/update from guide; [Collection of Custom Formats](https://trash-guides.info/Sonarr/Sonarr-collection-of-custom-formats/).

## Naming

- Same rationale as Radarr: filenames should carry quality/source/edition so Sonarr can avoid duplicate downloads and re-imports. Use TRaSH recommended series format; full templates: [Sonarr recommended naming scheme](https://trash-guides.info/Sonarr/Sonarr-recommended-naming-scheme/).

## Tips

- **Remote Path Mappings** — When Sonarr and download client see different paths (e.g. Docker).
- Language Custom Formats, merge quality, order quality sources, rename root folder, sync multiple instances: [Sonarr Tips](https://trash-guides.info/Sonarr/).

---
> Source: [amazor/Self-Hosting](https://github.com/amazor/Self-Hosting) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-29 -->
