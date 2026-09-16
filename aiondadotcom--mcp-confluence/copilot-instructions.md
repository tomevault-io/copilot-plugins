## mcp-confluence

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Development Commands

### Build and Development
```bash
# Build the project
npm run build

# Development mode with hot reload
npm run dev

# Start the built server
npm run start
```

### Testing and Quality
```bash
# Run tests
npm test

# Run tests in watch mode
npm test:watch

# Lint TypeScript files
npm run lint

# Format code
npm run format
```

### MCP Server Operations
```bash
# Start the MCP server (production)
./start-server.sh

# The startup script automatically:
# - Installs dependencies if node_modules missing
# - Builds the project if dist directory missing
# - Starts the server with node dist/index.js
```

## Project Architecture

### Core Components

**MCP Server (`src/server.ts`)**
- Main MCP server class implementing the Model Context Protocol
- Handles tool registration, resource management, and request routing
- Manages Confluence client lifecycle and configuration state
- Implements 9 tools and 3 resources for Confluence API access

**Confluence Client (`src/confluence/client.ts`)**
- Axios-based HTTP client for Confluence REST API
- Handles authentication, rate limiting, and error recovery
- Implements automatic token refresh and retry logic
- Supports CQL queries, page operations, space management, and write operations
- Enhanced error handling with detailed error messages and debugging information
- Improved page update functionality with version conflict detection
- Comprehensive validation for page update operations

**Configuration System (`src/config/index.ts`)**
- Zod-based configuration validation and management
- Stores credentials in `config.json` (git-ignored)
- Supports interactive setup and token renewal
- Handles configuration persistence and validation

### Key Features

**Authentication Flow**
- Initial setup requires manual configuration via `setup_confluence` tool
- API token-based authentication with Atlassian Cloud
- Automatic token validation and renewal prompts
- Configuration stored locally in `config.json`

**MCP Integration**
- Implements MCP SDK for STDIO transport
- Provides 9 tools: search, page/space operations, setup, create/update pages
- Exposes 3 resources: spaces, recent pages, user info
- Error handling with user-friendly English messages
- Robust error recovery with detailed debugging information
- Version conflict detection for concurrent page modifications

**API Operations**
- CQL-based content search with flexible queries
- Page retrieval with expandable fields
- Space listing and information retrieval
- Recent pages tracking and full-text search
- Page creation and updates with Atlassian Markup Format validation

**Write Operations with Format Validation**
- Create and update Confluence pages
- Automatic detection of Markdown syntax with helpful error messages
- Strong warnings about Atlassian Markup Format requirements
- Examples provided for proper format usage
- Enhanced page update with complete field validation (space, title, body)
- Version conflict detection and resolution guidance
- Comprehensive error reporting for debugging failed operations

### Project Structure

```
src/
├── index.ts          # Entry point and server startup
├── server.ts         # Main MCP server implementation
├── confluence/       # Confluence API integration
│   ├── client.ts     # HTTP client and API operations
│   ├── auth.ts       # Authentication handling
│   └── types.ts      # TypeScript type definitions
├── config/           # Configuration management
│   └── index.ts      # Config loading, validation, persistence
├── utils/            # Utility functions
│   └── errors.ts     # Custom error classes
├── tools/            # Tool implementations (if separate)
└── resources/        # Resource handlers (if separate)
```

### TypeScript Configuration

- ES2020 target with ESNext modules
- Strict TypeScript settings enabled
- Output to `dist/` directory
- Source maps and declarations generated
- ESM module system throughout

### Dependencies

**Runtime:**
- `@modelcontextprotocol/sdk`: MCP protocol implementation
- `axios`: HTTP client for Confluence API
- `zod`: Runtime type validation
- `fs-extra`: Enhanced file system operations

**Development:**
- TypeScript with strict configuration
- Jest for testing
- ESLint with TypeScript support
- Prettier for code formatting

## Configuration Notes

- Configuration is stored in `config.json` (git-ignored)
- Use `setup_confluence` tool for initial configuration
- Supports action types: `setup`, `update_token`, `validate`
- Rate limiting configured for 100 requests per minute
- All user-facing messages are in English

## Content Format Support

The MCP server now supports both Markdown and Atlassian Markup Format:

**Markdown Support (Automatically Converted):**
```markdown
# Heading
## Subheading
**bold text**
*italic text*
- bullet points
1. numbered lists
`inline code`
```

**Atlassian Markup Format:**
```
h1. Heading
h2. Subheading
*bold text*
_italic text_
* bullet points
# numbered lists
{{inline code}}
{info}Info box content{info}
{panel}Panel content{panel}
{code}code block{code}
```

The server automatically converts Markdown to Atlassian Markup Format when creating or updating pages.

## Error Handling & Debugging

The MCP server includes comprehensive error handling:

**Enhanced Error Reporting:**
- Detailed error messages with context and debugging information
- Specific error types for different failure scenarios (authentication, validation, network)
- Version conflict detection for concurrent page modifications
- Clear guidance for resolving common issues

**Debugging Features:**
- Comprehensive logging of API requests and responses
- Error context preservation for troubleshooting
- User-friendly error messages with actionable solutions
- Automatic retry logic for transient failures

**Common Error Scenarios:**
- Version conflicts when multiple users edit the same page
- Authentication failures with token expiration guidance
- Validation errors with specific field requirements
- Network connectivity issues with retry recommendations

---
> Source: [AiondaDotCom/mcp-confluence](https://github.com/AiondaDotCom/mcp-confluence) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-15 -->
