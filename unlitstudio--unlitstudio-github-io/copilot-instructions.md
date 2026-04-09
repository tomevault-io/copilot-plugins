## unlitstudio-github-io

> Static content repository hosted on GitHub Pages at `unlitstudio.github.io`. Serves as a public distribution point for configuration data and assets consumed by the desktop applications.

# Unlit Studio GitHub Pages — Static Content Repository

Static content repository hosted on GitHub Pages at `unlitstudio.github.io`. Serves as a public distribution point for configuration data and assets consumed by the desktop applications.

## Purpose

- Hosts **encrypted configuration** (URLs, version info) fetched by the prelauncher
- Provides **content metadata** (brand/chunk data) for the main launcher
- Serves **public images** used in the dashboard and launcher UI

## Structure

```
unlit-studio-github/
├── index.html                          # Landing page
└── Public/
    ├── Data/
    │   ├── Links.txt                   # AES-256 encrypted config URLs (fetched by prelauncher)
    │   ├── ChunkAndBrand.json          # Chunk-to-brand mapping data
    │   └── Content.json                # Content metadata
    └── Images/
        └── Homepage_*.jpg              # Brand/homepage images
```

## How It's Used

1. **Prelauncher** fetches `https://unlitstudio.github.io/Public/Data/Links.txt`, decrypts with AES-256 ECB to get config URLs (version check, download links)
2. **Main launcher** reads content metadata from `Public/Data/` for chunk/brand information
3. **Dashboard** may reference images from `Public/Images/`

## Key Details

- `Links.txt` is **AES-256 ECB encrypted** — the decryption key is in `AesOperation.cs` (both prelauncher and main launcher)
- This is a **separate Git repo** from the main launcher repository
- Changes here affect what the prelauncher and launcher fetch at runtime — update carefully

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/UnlitStudio)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/UnlitStudio)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
