## pywa-mcp-server

> This is a **PyWA MCP Server** - a Model Context Protocol (MCP) server that exposes comprehensive WhatsApp Business API functionality through PyWA (Python WhatsApp framework). The server provides 40+ WhatsApp tools for messaging, media, interactive content, templates, and more.

# PyWA MCP Server - Project Understanding

## Project Overview

This is a **PyWA MCP Server** - a Model Context Protocol (MCP) server that exposes comprehensive WhatsApp Business API functionality through PyWA (Python WhatsApp framework). The server provides 40+ WhatsApp tools for messaging, media, interactive content, templates, and more.

## Architecture & Implementation

### Core Structure
- **server.py**: Main MCP server entry point using FastMCP framework
- **tools/**: Modular tool implementations organized by functionality
  - **messaging.py**: Text, media, location, contact, reactions (17 tools)
  - **interactive.py**: Buttons, lists, catalogs, products, flows (9 tools)  
  - **templates.py**: Template messages, authentication, carousels (6 tools)
- **main.py**: Simple hello-world placeholder

### Key Implementation Details
- Uses **pywa>=3.0.0** - production-ready WhatsApp Cloud API library
- **FastMCP** for MCP server implementation with async support
- **Modular design** - each tool category in separate files
- **Direct API mapping** - tools map 1:1 to PyWA methods
- **Consistent error handling** - all tools return {"success": bool, "error": str} format
- **Environment-based config** - requires WHATSAPP_PHONE_ID and WHATSAPP_TOKEN
- **Outbound only** - no webhooks or inbound message handling

### PyWA Framework Understanding
- PyWA is the leading Python library for WhatsApp Business Cloud API
- Provides both sync and async clients (we use sync)
- Handles authentication, media upload, template management
- Supports all WhatsApp message types: text, media, interactive, templates
- Has comprehensive type system and error handling
- Production-ready with proper rate limiting and retry logic

## Environment & Setup

### Required Environment Variables
```bash
WHATSAPP_PHONE_ID=your_phone_number_id
WHATSAPP_TOKEN=your_whatsapp_cloud_api_token
```

### Installation & Running
```bash
# Install dependencies
uv sync

# Run the server
uv run python server.py
```

### Dependencies
- **mcp>=1.13.1** - Model Context Protocol implementation
- **pywa>=3.0.0** - WhatsApp Cloud API client
- **python-dotenv** - Environment variable loading

## Testing

### Web UI Testing (Recommended)
The fastest way to test and debug WhatsApp tools:

```bash
# Start development server with web interface
uv run fastmcp dev --ui-port 6275 server.py
```

**Access at:** http://localhost:6275

**Features:**
- **Interactive tool testing** - Fill forms and call WhatsApp tools directly
- **Real-time responses** - See API calls and responses immediately  
- **Error debugging** - Clear error messages with stack traces
- **Parameter validation** - Test constraint checking before sending
- **Authentication token** - Secure access with session tokens

### Claude Desktop Testing
After setting up MCP integration, test with natural language:

```
Send a WhatsApp message to +1234567890 saying "Hello from Claude!"
```

```
Create a WhatsApp message with Yes/No buttons asking "Are you available?" to +1234567890
```

### Manual Testing
For direct API testing without UI:

```bash
# Run linting (if available)
ruff check .

# Type checking (if available) 
mypy server.py tools/

# Production server testing
uv run python server.py
```

### Testing Approach
- **Web UI first** - Use FastMCP Inspector for development and debugging
- **Claude Desktop integration** - Test natural language interactions
- **Real WhatsApp API** - All tools connect to actual WhatsApp Business API
- **No mock testing** - Direct integration with PyWA library and WhatsApp

## Available Tools Summary

### Messaging Tools (12)
- **Text & Media**: send_message, send_image, send_video, send_document, send_audio, send_sticker
- **Location & Contacts**: send_location, request_location, send_contact
- **Interactions**: send_reaction, remove_reaction, upload_media

### Interactive Tools (2) 
- **Buttons & Lists**: send_message_with_buttons, send_message_with_list

### Template Tools (2)
- **Templates**: send_template, get_templates

### Status Tools (2)
- **Message Status**: mark_message_as_read, indicate_typing

**Total: 18 WhatsApp tools**

## Development Guidelines

### Adding New Tools
1. Create tool function in appropriate module (messaging/interactive/templates)
2. Use @mcp.tool() decorator with proper docstring
3. Follow consistent error handling pattern
4. Add to module's register_*_tools() function
5. Import and register in tools/__init__.py

### Code Conventions
- **Async functions** - all tools are async
- **Type hints** - comprehensive typing from PyWA
- **Logging** - use module-level logger
- **Error handling** - try/except with consistent response format
- **Documentation** - detailed docstrings with Args/Returns

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Jem-HR) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
