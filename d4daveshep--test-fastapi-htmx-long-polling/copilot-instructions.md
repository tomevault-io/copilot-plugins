## test-fastapi-htmx-long-polling

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a FastAPI + HTMX long-polling demonstration project that shows real-time web updates without WebSockets or server-sent events. The project contains two separate applications demonstrating different long-polling approaches.

## Architecture

### Core Applications

1. **main.py** - Primary notification system using timestamp-based polling
   - FastAPI server with Jinja2 templates
   - In-memory notifications storage with 50-item limit
   - 30-second timeout long-polling endpoint `/poll`
   - Background task generates notifications every 10-30 seconds
   - Manual notification creation via `/add-notification`

2. **test_long_polling.py** - Event-driven polling system for multiple users
   - Uses asyncio.Event for immediate notification of new events
   - Supports multiple users (alice, bob) with separate event streams
   - 60-second timeout with proper cleanup
   - EventGenerator class handles background event creation

### Template Structure

- `index.html` - Main UI for notification system (main.py)
- `home_polling.html` - Simple UI for event polling (test_long_polling.py)
- `notifications.html` - Fragment template for new notifications
- `poll_events_fragment.html` - Fragment template for event updates
- `status.html` - System status display

### Long-Polling Architecture

Both applications implement long-polling where:
- Client sends request with last known event/timestamp
- Server waits (30-60 seconds) for new data
- Returns immediately when new data arrives
- Client automatically triggers next polling request via HTMX

## Development Commands

Start the main notification system:
```bash
python main.py
# Runs on http://localhost:8000
```

Start the event polling test system:
```bash
python test_long_polling.py
# Runs on http://localhost:8001
```

Install dependencies:
```bash
uv sync
```

## Key Technical Details

- **In-memory storage**: Both apps use in-memory data structures (not persistent)
- **HTMX integration**: Pure declarative HTMX with no custom JavaScript
- **Async/await**: Proper asyncio usage for non-blocking operations
- **Background tasks**: FastAPI startup events launch background notification generators
- **Memory management**: Automatic cleanup of old notifications/events
- **Error handling**: Timeout handling and proper cleanup of event notifiers

## Testing

The project includes `test_long_polling.py` which serves as both a test application and an alternative implementation approach using asyncio events instead of timestamp polling.

Both applications can run simultaneously on different ports for comparison.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/d4daveshep)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/d4daveshep)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
