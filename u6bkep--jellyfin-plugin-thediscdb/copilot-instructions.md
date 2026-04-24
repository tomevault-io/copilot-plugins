## jellyfin-plugin-thediscdb

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Jellyfin plugin that resolves BDMV (Blu-ray) folders into individual episodes using playlist-to-episode mappings from a local clone of the TheDiscDb data repository (https://github.com/TheDiscDb/data). When Jellyfin scans a TV show library and encounters a BDMV folder inside a Season directory, this plugin intercepts the scan, computes a content hash from the disc's `.m2ts` files, looks up the disc in the local index, and creates individual Episode items pointing to the `.mpls` playlist files.

## Build

```bash
dotnet build
```

The plugin targets `net9.0` and depends on `Jellyfin.Controller` and `Jellyfin.Model` 10.11.6 (compile-only, excluded at runtime). No solution file — build directly from the `.csproj`.

## Deploy to Test Jellyfin

Build the DLL and copy it into the test environment:

```bash
dotnet build
cp bin/Debug/net9.0/Jellyfin.Plugin.TheDiscDb.dll test/config/plugins/TheDiscDb/
```

The `test/` directory contains a full Jellyfin server config (database, library root, plugin config) for manual testing. The media library root is at `test/media/TV/`.

## Architecture

The plugin has four key components that form a pipeline:

1. **ContentHashCalculator** (`TheDiscDb/ContentHashCalculator.cs`) — Computes an MD5 hash from the sorted `.m2ts` file sizes in `BDMV/STREAM/`. This hash is TheDiscDb's disc fingerprint format (little-endian Int64 sizes, sorted by filename).

2. **TheDiscDbClient** (`TheDiscDb/TheDiscDbClient.cs`) — Reads disc metadata from a local clone of the TheDiscDb/data git repo. Builds an in-memory `Dictionary<string, DiscNode>` keyed by ContentHash at startup by scanning all `disc*.json` files. Lookups are synchronous and instant.

3. **BdmvEpisodeResolver** (`Resolvers/BdmvEpisodeResolver.cs`) — Implements both `IItemResolver` and `IMultiItemResolver`. Two entry points:
   - `ResolvePath`: Intercepts BDMV-containing folders under a Series to return a `Season` (preventing Jellyfin's built-in EpisodeResolver from claiming the whole BDMV as one episode).
   - `ResolveMultiple`: When Jellyfin scans the Season's children, splits the BDMV into individual `Episode` and `Video` (extras) items. Each episode's Path points to the `.mpls` playlist file for unique identity.

4. **BdmvEpisodePreRefreshProvider** (`Providers/BdmvEpisodePreRefreshProvider.cs`) — `ICustomMetadataProvider<Episode>` that runs after all other metadata providers. Restores correct season/episode numbering from the `TheDiscDb` provider ID string (format: `hash:playlist:SxxExx:Title`), as a safety net for "Replace all metadata" scenarios.

**MplsParser** (`Parsers/MplsParser.cs`) — Minimal binary parser for Blu-ray `.mpls` playlist files. Extracts clip filenames from PlayItem entries. Not currently used by the resolver but available for future use.

**Models** (`TheDiscDb/Models.cs`) — DTOs matching the TheDiscDb data repo JSON format (PascalCase property names). Disc files contain ContentHash, Titles with SourceFile/SegmentMap/Item metadata.

## Key Design Decisions

- Data comes from a local git clone of TheDiscDb/data — no network calls during scans. Users `git pull` to update.
- Episode paths point to `.mpls` playlist files, giving each episode a unique Jellyfin item ID and letting ffprobe/ffmpeg handle segment resolution.
- The Season is NOT locked — this allows TMDB to enrich episodes with titles/descriptions while the custom provider preserves correct numbering.
- Episodes encode all TheDiscDb metadata in the provider ID string so the PreRefreshProvider can restore it without re-querying.
- Non-BDMV files in a Season folder are passed through as `ExtraFiles` so normal resolvers still handle them.
- `ImplicitUsings` is disabled — all `using` statements are explicit.

## TheDiscDb Data Repo Structure

```
data/
  series/
    Show Name (Year)/
      metadata.json          # Title, Slug, ExternalIds (TMDB, IMDB)
      2015-blu-ray/          # One dir per physical release
        release.json
        disc01.json          # ContentHash + Titles[] with playlist mappings
        disc02.json
  movie/
    ...
  sets/
    ...
```

## Reference Material

- The Jellyfin codebase is available at `/home/ben/Documents/programmingPlay/jellyfin`
- The TheDiscDb data repo: https://github.com/TheDiscDb/data

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/u6bkep) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
