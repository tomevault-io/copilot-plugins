## rd-dl

> A CLI tool for downloading bookmarks from Raindrop.io using their REST API.

# rd_dl - Raindrop.io Download Tool

## Project Overview
A CLI tool for downloading bookmarks from Raindrop.io using their REST API.

## API Reference
- Documentation: https://developer.raindrop.io
- Authentication: Access token required
- Main endpoints:
  - Collections: Manage bookmark collections
  - Raindrops: Manage individual bookmarks
  - User, Tags, Filters, Import/Export

## Usage
- `rd_dl` - Download bookmarks from Raindrop.io
- `rd_dl --collection <id>` - Download from specific collection
- `rd_dl --format <json|csv|html>` - Export format

## Implementation
- Python CLI tool using uv for dependency management
- Uses requests library for API calls
- Handles authentication with access tokens
- Supports multiple export formats

## Specification
* Tool will pull down all the "raindrops" from my raindrop.ai
* It will remember what it has downloaded so it never downloads the same one again, just the new ones
* It is designed to be idempotent. If I delete the output and run it again I should get the exact same output
* The results are placed in a directory called drops
* The output is reformatted into markdown files
* They are called drop-yy-dd-mm-n.md
* The markdown will contain yaml front matter for title, date, tags
* They will also contain yaml for: "type: drop"
* 

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/pitosalas)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/pitosalas)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
