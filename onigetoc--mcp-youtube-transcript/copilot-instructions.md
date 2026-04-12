## mcp-youtube-transcript

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is an MCP (Model Context Protocol) server for extracting YouTube video transcripts. It provides transcript extraction capabilities via the MCP protocol and includes both MCP-compatible server functionality and a standalone CLI for direct usage.

## Technology Stack

- **Runtime**: Node.js with TypeScript
- **Build System**: `tsc` (TypeScript compiler)
- **Dependencies**: 
  - `@modelcontextprotocol/sdk`: MCP protocol implementation
  - `@danielxceron/youtube-transcript`: YouTube transcript extraction
  - `zod`: Schema validation
- **Package Manager**: npm (also compatible with bun)

## File Structure

```
src/
├── index.ts      # Main MCP server implementation
├── cli.ts        # Standalone CLI tool for transcript extraction

build/            # Compiled JavaScript output (generated)
test-client.cjs   # Legacy JSON-RPC test client (non-functional)
```

## Development Commands

### Build & Development
```bash
npm install        # Install dependencies
npm run build      # Compile TypeScript to JavaScript
tsc --watch        # Watch mode for development
```

### Testing & Usage
```bash
# MCP Server (stdio-based)
npm start

# TypeScript development mode
npm run dev        # Run via tsx for development

# CLI Usage (after build)
node build/cli.js [youtube-url] [--lang en,fr] [--timestamps] [--max 6000]
node build/cli.js https://youtu.be/JIUI6O5pSas --lang=en,fr --timestamps --max=4000

# Default demo (no args)
node build/cli.js  # Uses demo video
```

## Architecture

### Core Functions

The server exposes two MCP tools:

1. **`youtube_transcript`** - Extracts transcripts with options
   - Parameters: `url` or `video_id`, `languages[]`, `include_timestamps`, `max_chars_per_chunk`
   - Returns: Text chunks for large transcripts

2. **`youtube_available_languages`** - Detects available subtitle languages
   - Parameters: `url` or `video_id`
   - Returns: Array of language codes

### Key Utilities

- **Video ID extraction**: Supports full URLs (`youtube.com/watch?v=...`, `youtu.be/...`) or direct 11-char IDs
- **Language fallback**: Tries user-specified languages, falls back to auto-detect
- **Text cleaning**: Decodes HTML entities, normalizes encoding glitches, handles musical symbols
- **Chunking**: Manages large transcripts by character limit (configurable 500-20000 chars)
- **Timestamp formatting**: `[mm:ss]` or `[hh:mm:ss]` format based on segment start times

### CLI Parameters

The CLI uses these flag patterns:
- `--lang en,fr` - Comma-separated language priorities
- `--timestamps` - Include timestamps in output
- `--max 6000` - Max characters per chunk (0 = no chunking)
- No URL argument = uses demo video

## Integration

### Claude Desktop Configuration

```json
{
  "mcpServers": {
    "gc-youtube-transcript": {
      "command": "node",
      "args": ["C:/absolute/path/to/build/index.js"]
    }
  }
}
```

## Environment Requirements

- No API keys required - uses open transcript extraction library
- Node.js 18+ recommended
- TypeScript compilation necessary before runtime usage
- Executes as stdio-based MCP server or direct Node CLI

## Build Output

Executable files generated in `build/`:
- `index.js` - MCP server entry point
- `cli.js` - Standalone CLI executable

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/onigetoc)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/onigetoc)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
