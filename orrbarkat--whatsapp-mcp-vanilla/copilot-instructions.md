## whatsapp-mcp-vanilla

> This file provides guidance to WARP (warp.dev) when working with code in this repository.

# WARP.md

This file provides guidance to WARP (warp.dev) when working with code in this repository.

Project scope
- Two components: a Go WhatsApp bridge (whatsapp-bridge/) and a Python MCP server (whatsapp-mcp-server/).
- The Go bridge links your personal WhatsApp via whatsmeow, persists data in SQLite, and exposes a small REST API.
- The Python MCP server implements Model Context Protocol tools that query the SQLite DB for reads and call the Go REST API to send/download media.

Essential commands
- Start the WhatsApp bridge (first-run will print a QR code for linking):
  - macOS/Linux:
    - go run ./whatsapp-bridge/main.go
  - Windows (CGO required for go-sqlite3):
    - go env -w CGO_ENABLED=1
    - go run ./whatsapp-bridge/main.go

- Build a standalone binary for the bridge:
  - mkdir -p bin && go build -o bin/whatsapp-bridge ./whatsapp-bridge

- Reset local WhatsApp session/data (forces re-auth and re-sync):
  - rm -f whatsapp-bridge/store/messages.db whatsapp-bridge/store/whatsapp.db

- Python MCP server (uses uv, no manual venv needed):
  - First-time install (optional; uv can also resolve on the fly):
    - uv sync --directory whatsapp-mcp-server
  - Run the MCP server directly:
    - uv run --directory whatsapp-mcp-server main.py

- REST API debugging for the bridge (useful without MCP):
  - Send a text message:
    - curl -X POST http://localhost:8080/api/send \
      -H 'Content-Type: application/json' \
      -d '{"recipient":"{{recipient_or_jid}}","message":"Hello from bridge"}'
  - Trigger media download for a stored message:
    - curl -X POST http://localhost:8080/api/download \
      -H 'Content-Type: application/json' \
      -d '{"message_id":"{{message_id}}","chat_jid":"{{chat_jid}}"}'

Claude/Cursor integration (summary)
- Configure your client to launch the MCP server via uv:
  - Command should be the full path to uv (run which uv to get it).
  - Args: ["--directory","<repo>/whatsapp-mcp-server","run","main.py"].
- Save the JSON to:
  - Claude Desktop: ~/Library/Application Support/Claude/claude_desktop_config.json
  - Cursor: ~/.cursor/mcp.json
- Restart Claude/Cursor after saving. See README.md in this repo for the full example JSON.

Architecture overview
- Data flow
  1) Claude (or Cursor) invokes MCP tools implemented in whatsapp-mcp-server/main.py via FastMCP.
  2) Read paths: Python code queries SQLite directly at whatsapp-bridge/store/messages.db for chats/messages and formats results for the tools (see whatsapp-mcp-server/whatsapp.py).
  3) Write/media paths: Python calls the Go bridge REST endpoints:
     - POST /api/send to send text or media; bridge uploads media to WhatsApp and sends the message.
     - POST /api/download to fetch media for a specific message using stored keys and whatsmeow’s downloader.
  4) The Go bridge uses whatsmeow to maintain a session, listens to events, and writes chats/messages (and media metadata) into SQLite.

- Storage and schema
  - SQLite files live under whatsapp-bridge/store/ (messages.db and whatsapp.db).
  - messages table stores: id, chat_jid, sender, content, timestamp, is_from_me, media_type, filename, plus fields needed for media download (URL, media_key, hashes, lengths).
  - chats table stores: jid, name, last_message_time.

- Components and responsibilities
  - Go (whatsapp-bridge/):
    - Authentication via QR code; reconnects on subsequent runs.
    - Event handlers persist history and live messages; REST server listens on :8080.
    - Media handling: uploads on send; downloads on request using saved metadata.
  - Python (whatsapp-mcp-server/):
    - Exposes MCP tools: search_contacts, list_messages (+context), list_chats, get_chat, get_direct_chat_by_contact, get_contact_chats, get_last_interaction, get_message_context, send_message, send_file, send_audio_message, download_media.
    - Audio conversion helper (audio.py) uses ffmpeg to convert non-.ogg audio to Opus .ogg for WhatsApp voice messages.

Notes for development
- Ensure the Go bridge is running before relying on MCP tools that send/download media (Python’s send_* and download_media call the bridge’s REST API).
- For voice messages, install ffmpeg so audio.py can convert inputs to Opus .ogg automatically; otherwise fallback to send_file.
- Windows only: enable CGO and install a C compiler (e.g., MSYS2) before running/building the bridge (see README.md for links).

Tests and linting
- No test suite or lint configuration is present in this repository at the time of writing. If tests are added later, prefer documenting how to run a single test here.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/orrbarkat) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
