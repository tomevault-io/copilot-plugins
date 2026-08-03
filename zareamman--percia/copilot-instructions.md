## percia

> TMDB Importer is a WordPress plugin that imports and synchronizes Movies, TV Shows, Seasons, Episodes, metadata, taxonomies, images and videos from TMDB.

# CLAUDE.md — TMDB Importer (Product Specification)

## Overview
TMDB Importer is a WordPress plugin that imports and synchronizes Movies, TV Shows, Seasons, Episodes, metadata, taxonomies, images and videos from TMDB.

### Core Principles
- Fast imports
- SEO-friendly content
- WordPress-first user experience
- Automated synchronization
- Flexible media handling
- Theme-friendly output

## Product Goals
- Import complete TMDB content.
- Keep content synchronized.
- Prevent duplicates.
- Preserve user changes where possible.
- Provide extensive configuration.

## Content Model
### Movies
Metadata, images, ratings, videos, taxonomies.

### TV Shows
Metadata, seasons, episodes, ratings, videos, taxonomies.

### Seasons
Season number, poster, overview, air date, episode count.

### Episodes
Episode number, still image, overview, runtime, air date.

## Supported Taxonomies
- Genres
- Release Year
- Keywords
- People
- Networks
- Production Companies

Each taxonomy supports archives, filtering, metadata, TMDB ID and optional images.

## Image Management
Default: store TMDB URLs.
Optional: download to Media Library.

Supported:
- Posters
- Backdrops
- Logos
- Stills
- People images

## Video Management
Default:
- Store YouTube IDs

Optional:
- Download trailers

## Synchronization
Supports updating:
- Ratings
- Metadata
- Images
- Videos
- Taxonomies
- Seasons
- Episodes

## Settings
### API
- API Key
- Language
- Region

### Import
- Skip existing
- Overwrite
- Import missing seasons
- Import missing episodes

### Images
- Store URLs
- Download images
- Image size
- Replace existing

### Videos
- Store YouTube IDs
- Download trailers

### Maintenance
- Clear cache
- Remove orphaned content
- Reset settings

## UX
- Simple import UI
- Progress indicators
- Background processing
- Clear validation
- Theme-friendly output

## Design Principles
- WordPress-first
- Performance-first
- SEO-first
- Extensible
- Backward compatible

---
> Source: [zareamman/percia](https://github.com/zareamman/percia) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-25 -->
