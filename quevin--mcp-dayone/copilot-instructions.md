## mcp-dayone

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

This is a comprehensive MCP (Message Control Protocol) server that provides full read/write integration between Claude Desktop and Day One Journal. The server implements the MCP protocol with 10 specialized tools for creating, reading, and searching journal entries with rich metadata support.

## Project Status - Production Ready

**Current Phase**: Full-featured production deployment ✅  
**Progress**: Complete read/write functionality with advanced features

### Recent Completed Features:
- ✅ **Enhanced Text Extraction**: Comprehensive rich text JSON parsing from Day One's complex format
- ✅ **"On This Day" Feature**: Date-based historical entry retrieval across multiple years
- ✅ **Direct Database Access**: SQLite integration for reading Day One's Core Data database
- ✅ **Hybrid Architecture**: CLI for writing, database for reading operations
- ✅ **10 MCP Tools**: Complete toolkit for journal management
- ✅ **Advanced Attachments**: Support for photos, videos, audio, PDFs (up to 10 per entry)
- ✅ **Location Integration**: GPS coordinates and location-aware entries
- ✅ **Rich Metadata**: Tags, starred entries, timezone support, all-day events

## Architecture

### Production Python MCP Structure:
- **src/mcp_dayone/server.py**: Main MCP server with 10 tool handlers and proper error handling
- **src/mcp_dayone/tools.py**: Hybrid Day One operations:
  - **CLI wrapper** for write operations (create_entry, attachments, location)
  - **SQLite database access** for read operations (Core Data schema)
  - **Text extraction engine** for Day One's rich text JSON format
- **pyproject.toml**: Python project dependencies and configuration
- **docs/**: Documentation and examples

### Key Implementation Details:
- **Database Path**: `~/Library/Group Containers/5U8NS4GX82.dayoneapp2/Data/Documents/DayOne.sqlite`
- **Core Data Schema**: Uses Z-prefixed tables (ZENTRY, ZJOURNAL, ZTAG, Z_13TAGS)
- **Timestamp Conversion**: Core Data timestamps (seconds since 2001-01-01) + 978307200
- **Rich Text Parsing**: Handles multiple JSON formats (attributedString, ops, delta, NSString)

## Common Commands

### Development (New Python MCP)
- `uv sync` - Install dependencies and create virtual environment
- `uv run python -m mcp_dayone.server` - Run MCP server directly
- `uv run python test_setup.py` - Test installation and Day One CLI
- `uv run pytest` - Run tests (when implemented)

### Getting Started
1. Install Day One app and CLI from [Day One CLI Guide](https://dayoneapp.com/guides/tips-and-tutorials/command-line-interface-cli)
2. Verify CLI access: `dayone2 --version`
3. Install project: `uv sync`
4. Test setup: `uv run python test_setup.py`
5. Configure Claude Desktop with absolute path to this project

### Legacy Commands (Node.js - removed)
- ~~`npm start` - Start old REST server~~ (removed)
- ~~`npm run dev` - Start with nodemon~~ (removed)

## Dependencies

### Required System Dependencies
- **Day One CLI** (`dayone2`) - Install from [Day One CLI Guide](https://dayoneapp.com/guides/tips-and-tutorials/command-line-interface-cli). Must be accessible in PATH.
- **Python 3.11+** 
- **uv package manager** - Install from [astral.sh](https://astral.sh/uv/install.sh)

### Python Dependencies
- mcp>=1.0.0 - MCP protocol implementation
- click>=8.0.0 - CLI utilities
- pydantic>=2.0.0 - Data validation

## Complete MCP Tools (10 Total)

### 📝 **Write Operations (CLI-based)**
1. **create_journal_entry** - Full-featured entry creation with all metadata:
   - Content, tags, date, journal selection
   - Attachments: photos, videos, audio, PDFs (max 10 per entry)
   - Location: GPS coordinates (latitude/longitude)
   - Metadata: starred flag, timezone, all-day events
2. **create_entry_with_attachments** - Specialized for file attachment workflows
3. **create_location_entry** - Specialized for location-aware journaling

### 📖 **Read Operations (Database-based)**
4. **read_recent_entries** - Recent entries with full text extraction and metadata
5. **search_entries** - Full-text search across entry content with context
6. **get_entries_by_date** - **"On This Day" feature** - Historical entries by date across years
7. **list_journals_from_db** - Real journal listing with entry counts and statistics
8. **get_entry_count_from_db** - Accurate entry counts from database

### 📋 **Legacy Tools (CLI limitations)**
9. **list_journals** - Guidance about CLI limitations for journal listing
10. **get_entry_count** - Explains CLI counting limitations

### Key Features:
- **Enhanced Text Extraction**: Robust parsing of Day One's rich text JSON format
- **"On This Day" Memories**: Date-based retrieval across multiple years (e.g., "06-14" searches June 14th in past 5 years)
- **Full Metadata Support**: Tags, starred status, locations, weather, attachments
- **Error Resilience**: Comprehensive error handling and graceful degradation

## Claude Desktop Integration

The server uses MCP protocol over stdio. Configure in `claude_desktop_config.json`:

```json
{
  "mcpServers": {
    "dayone": {
      "command": "uv",
      "args": ["--directory", "/FULL/PATH/TO/mcp-dayone", "run", "python", "-m", "mcp_dayone.server"]
    }
  }
}
```

**Important**: Replace `/FULL/PATH/TO/mcp-dayone` with absolute path to this repository.

## Common Usage Patterns

### Natural Language Queries Supported:
- **"Create a journal entry about my day"** → Uses create_journal_entry
- **"Show me my recent journal entries"** → Uses read_recent_entries  
- **"Search my journal for entries about work"** → Uses search_entries
- **"What were my journal entries on this day?"** → Uses get_entries_by_date with today's date
- **"Show me entries from June 14th in past years"** → Uses get_entries_by_date with "06-14"
- **"List my Day One journals"** → Uses list_journals_from_db
- **"Create entry with photos from /path/to/image.jpg"** → Uses create_entry_with_attachments

### Technical Notes for Development:
- All read operations use direct SQLite database access for performance
- Text extraction handles multiple Day One rich text JSON formats automatically  
- Date parsing supports MM-DD, YYYY-MM-DD, and natural language formats
- Error handling provides helpful user feedback for common issues (missing CLI, database access, etc.)
- Database path is automatically detected for standard Day One installations

---
> Source: [Quevin/mcp-dayone](https://github.com/Quevin/mcp-dayone) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-20 -->
