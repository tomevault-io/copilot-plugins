## apple-podcast-downloader

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Claude Code skill for searching, browsing, and downloading podcast episodes from Apple Podcasts using the iTunes Search API. It's designed as a zero-dependency Python tool using only the standard library.

## Architecture

### Core Components

**`SKILL.md`** - The main skill definition that Claude Code reads to understand how to use this skill. Contains:
- Command patterns and workflows
- Data structures and API reference
- Best practices for user experience

**`scripts/itunes_api.py`** - Python helper script that provides CLI interface to iTunes API with three main commands:
- `search <keyword> [limit]` - Search for podcasts
- `episodes <collection_id> [limit]` - List episodes from a podcast
- `download <collection_id> <episode_index> [output_dir]` - Download episode audio

### Key Design Patterns

**Output Separation**: The helper script separates data from progress messages:
- JSON data → stdout (for Claude to parse)
- Progress/status → stderr (for user visibility)

**Zero-Based Indexing**: Episodes are indexed from 0 (newest episode = index 0)

**Safe Filenames**: Automatic sanitization of podcast/episode names for filesystem compatibility with Unicode support

**Streaming Downloads**: Large audio files are downloaded in chunks with progress tracking

## Common Development Commands

### Testing the Helper Script

```bash
# Search for podcasts
python3 scripts/itunes_api.py search "python programming" 5

# Get episodes from a podcast (ID from search results)
python3 scripts/itunes_api.py episodes 979020229 10

# Download an episode (index 0 = latest)
python3 scripts/itunes_api.py download 979020229 0 downloads/
```

### Using as a Claude Skill

The skill is invoked automatically when users ask about podcasts. No manual commands needed - Claude reads `SKILL.md` and knows how to:
1. Search for podcasts by keyword
2. Display episodes with metadata
3. Download audio files to organized directories

## Important Implementation Details

### iTunes API Behavior

**Search Results**: Returns podcast metadata including `collectionId` (primary key for subsequent operations)

**Episode Fetching**: API limit is ~100 episodes per request. For older episodes, use the RSS feed URL from search results.

**Episode List Structure**: First result from `/lookup` endpoint is always the podcast itself; actual episodes start at index 1 in raw API response. The helper script filters this automatically.

### Data Flow

```
User Request
    ↓
Claude parses intent
    ↓
Execute helper script (search/episodes/download)
    ↓
Parse JSON output from stdout
    ↓
Present to user / download file
```

### Error Handling

**Network Errors**: Helper script catches URLError/HTTPError and outputs friendly messages to stderr

**Invalid IDs**: Returns empty results array with appropriate error message

**Download Failures**: Returns JSON with `{"success": false, "error": "..."}` for Claude to parse

### File Organization

Downloads are automatically organized as:
```
downloads/podcasts/
  ├── Podcast Name/
  │   ├── Podcast Name - Episode Title 1.mp3
  │   └── Podcast Name - Episode Title 2.mp3
  └── Another Podcast/
      └── ...
```

## Workflow Patterns

**Quick Download**: Search → Get latest episode → Download
- User: "Download latest episode of The Daily"
- Pattern: 1 search, 1 episode fetch (limit=1), 1 download

**Browse and Choose**: Search → List episodes → User selects → Download
- User: "Show me recent episodes of Python Bytes"
- Pattern: 1 search, 1 episode fetch (limit=10), display, wait for selection

**Batch Download**: Search → Get multiple episodes → Download all
- User: "Download 5 latest episodes of All-In"
- Pattern: 1 search, 1 episode fetch (limit=5), sequential downloads

## API Constraints

- **Rate Limit**: ~20 requests/second (unofficial, implement backoff on errors)
- **Search Results**: Max 200 per request
- **Episode List**: Practical limit ~100 per request via API
- **No Authentication**: Free public API, no keys required
- **Geographic Content**: Some podcasts may be region-restricted

## File Dependencies

- `SKILL.md` → Primary skill definition (auto-loaded by Claude)
- `examples.md` → 8 detailed usage examples with expected interactions
- `reference.md` → Complete API documentation and data structures
- `scripts/itunes_api.py` → Standalone Python script with no external dependencies

## Unicode Support

Full support for international characters in:
- Search keywords (Chinese, Japanese, Korean, etc.)
- Podcast names and episode titles
- File paths and names (sanitized for filesystem safety)

All string handling uses UTF-8 encoding throughout.

---
> Source: [crazynomad/apple-podcast-downloader](https://github.com/crazynomad/apple-podcast-downloader) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-25 -->
