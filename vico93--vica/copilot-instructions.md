## vica

> This document provides comprehensive guidance for Gemini and Antigravity to work on the Vica Discord bot project.

# Vica Discord Bot - AI Assistant Guide

This document provides comprehensive guidance for Gemini and Antigravity to work on the Vica Discord bot project.

## Table of Contents
- [Project Overview](#project-overview)
- [Architecture Overview](#architecture-overview)
- [Key Features](#key-features)
- [MCP Integration](#mcp-integration)
- [Database Schema](#database-schema)
- [Memory System](#memory-system)
- [Configuration](#configuration)
- [Code Patterns](#code-patterns)
- [Technical Details](#technical-details)
- [Development Notes](#development-notes)
- [File Structure](#file-structure)

---

## Project Overview

**Vica** is a Discord bot developed in Node.js that combines AI chatbot functionality with a local question bank system.

### Key Characteristics
- **Language**: Node.js (JavaScript)
- **Platform**: Discord.js library
- **Deployment**: Runs on Raspberry Pi 3b+ via systemd
- **AI Integration**: Uses OpenAI-compatible APIs (Requesty) for AI features
- **MCP Support**: Model Context Protocol for extending bot capabilities
- **Primary Language**: Portuguese (codebase and user-facing content)

### Purpose
Vica serves as an intelligent Discord assistant that can:
- Engage in AI-powered conversations with context awareness
- Manage user XP and ranking systems
- Store and retrieve long-term memories using semantic search
- Provide interactive configuration for server administrators
- Handle various administrative tasks through slash commands

---

## Architecture Overview

### Core Components

| Component | File | Purpose |
|-----------|------|---------|
| Database Interface | [`core/database.js`](core/database.js) | SQLite database operations with prepared statements |
| OpenAI Interface | [`core/oai_interface.js`](core/oai_interface.js) | AI API integration for chat and embeddings |
| MCP Client | [`core/mcp_client.js`](core/mcp_client.js) | Model Context Protocol client for external tools |
| Tag Parser | [`core/tagParser.js`](core/tagParser.js) | Parses special tags in messages ([salvar_memoria], [meta], [imagem]) |
| Tool Loader | [`core/tool_loader.js`](core/tool_loader.js) | Loads OpenAI function calling tools and MCP servers |
| Static Data | [`core/static_data.js`](core/static_data.js) | Manages static configuration data |
| Transcriber | [`core/transcriber.js`](core/transcriber.js) | Handles audio/image transcription |
| Audit Cache | [`core/auditCache.js`](core/auditCache.js) | Caches audit logs for performance |

### Commands (19 Slash Commands)

| Command | File | Purpose |
|---------|------|---------|
| `/canal_noticia` | [`commands/canal_noticia.js`](commands/canal_noticia.js) | Configure news channel |
| `/chat_blacklist` | [`commands/chat_blacklist.js`](commands/chat_blacklist.js) | Manage chatbot blacklist |
| `/comment` | [`commands/comment.js`](commands/comment.js) | Add comments to messages |
| `/deslurkar` | [`commands/deslurkar.js`](commands/deslurkar.js) | Remove user from blacklist |
| `/enex` | [`commands/enex.js`](commands/enex.js) | Enable/disable features |
| `/gmemories` | [`commands/gmemories.js`](commands/gmemories.js) | Manage guild memories |
| `/mmemories` | [`commands/mmemories.js`](commands/mmemories.js) | Manage user memories |
| `/noticia` | [`commands/noticia.js`](commands/noticia.js) | Post news with OpenGraph metadata |
| `/perguntar` | [`commands/perguntar.js`](commands/perguntar.js) | Ask questions to the AI |
| `/rank` | [`commands/rank.js`](commands/rank.js) | View user rank/XP |
| `/rank_blacklist` | [`commands/rank_blacklist.js`](commands/rank_blacklist.js) | Manage XP blacklist |
| `/rank_reset` | [`commands/rank_reset.js`](commands/rank_reset.js) | Reset user XP |
| `/rank_rolexp` | [`commands/rank_rolexp.js`](commands/rank_rolexp.js) | Set role XP multipliers |
| `/rank_setxp` | [`commands/rank_setxp.js`](commands/rank_setxp.js) | Set user XP manually |
| `/reaction_emoji` | [`commands/reaction_emoji.js`](commands/reaction_emoji.js) | Configure reaction emojis |
| `/reaction_thread` | [`commands/reaction_thread.js`](commands/reaction_thread.js) | Configure thread reactions |
| `/system_channel` | [`commands/system_channel.js`](commands/system_channel.js) | Configure system channel |
| `/trigger` | [`commands/trigger.js`](commands/trigger.js) | Set message triggers |
| `/up_role` | [`commands/up_role.js`](commands/up_role.js) | Configure role congratulations |

### Events (7 Discord Event Handlers)

| Event | File | Purpose |
|-------|------|---------|
| `guildBanAdd` | [`events/guildBanAdd.js`](events/guildBanAdd.js) | Handle user bans |
| `guildMemberAdd` | [`events/guildMemberAdd.js`](events/guildMemberAdd.js) | Welcome new members |
| `guildMemberRemove` | [`events/guildMemberRemove.js`](events/guildMemberRemove.js) | Handle member leaves |
| `guildMemberUpdate` | [`events/guildMemberUpdate.js`](events/guildMemberUpdate.js) | Track role changes |
| `interactionCreate` | [`events/interactionCreate.js`](events/interactionCreate.js) | Handle slash commands |
| `messageCreate` | [`events/messageCreate.js`](events/messageCreate.js) | Process messages for XP and chatbot |
| `messageReactionAdd` | [`events/messageReactionAdd.js`](events/messageReactionAdd.js) | Handle reaction-based features |

### Tools (2 OpenAI Function Calling Tools)

| Tool | File | Purpose |
|------|------|---------|
| `calculate` | [`tools/calculate.js`](tools/calculate.js) | Perform mathematical calculations |
| `get_current_time` | [`tools/get_current_time.js`](tools/get_current_time.js) | Get current date/time |

### MCP Servers (Model Context Protocol)

| Server | Transport | Status | Tools |
|--------|-----------|--------|-------|
| Memory | stdio | ✅ Working | 9 tools (knowledge graph operations) |
| Sequential Thinking | http | ⚠️ Needs investigation | - |
| Time | http | ⚠️ Needs investigation | - |

### Helpers

| Helper | File | Purpose |
|--------|------|---------|
| Embedding Helper | [`helpers/embeddingHelper.js`](helpers/embeddingHelper.js) | Semantic search using embeddings |

---

## Key Features

### 1. AI Chatbot with Conversation Context
- Maintains conversation history (last 6 messages)
- Supports multimodal input (text and images)
- Uses OpenAI-compatible API (Requesty)
- Context-aware responses
- Rate limiting: 5 seconds per user

### 2. XP/Ranking System
- Random XP gain per message (configurable min/max)
- Level-based progression
- Role-based XP multipliers
- Blacklist support for channels
- Manual XP adjustment commands
- Role congratulations on level up

### 3. Long-term Memory System
- User-specific memories
- Guild-wide memories
- Semantic search using embeddings
- Tag-based memory creation: `[salvar_memoria]`, `[meta]`, `[imagem]`
- Cosine similarity for memory retrieval

### 4. Server Configuration Commands
- Dedicated slash commands for server settings
- Ephemeral responses for admin commands
- Server-specific settings

### 5. Blacklist Systems
- Chatbot blacklist (channels where AI won't respond)
- XP blacklist (channels where XP is not awarded)
- User blacklist (users banned from chatbot)

### 6. Multimodal Support
- Image analysis via AI
- Audio transcription
- OpenGraph metadata extraction for news

### 7. News System
- Post news with rich embeds
- OpenGraph metadata extraction
- Configurable news channel
- Reaction-based thread creation

### 8. Role-based Congratulations
- Automatic congratulations on role assignment
- Customizable messages per role
- Server-specific configuration

### 9. Welcome/Leave Messages
- Customizable welcome messages
- Customizable leave messages
- Configurable system channel

---

## MCP Integration

### Overview

The Vica bot supports Model Context Protocol (MCP), a standardized protocol that enables AI assistants to communicate with external services and tools. MCP allows extending the bot's capabilities without modifying the core codebase.

### MCP Client Module

The [`core/mcp_client.js`](core/mcp_client.js) module provides the interface for MCP server communication:

**Key Functions:**
- `startServer(config)` - Initialize and start an MCP server
- `stopServer(server)` - Gracefully stop a running MCP server
- `initializeServer(server)` - Perform MCP handshake and initialization
- `listTools(server)` - Retrieve available tools from an MCP server
- `callTool(server, toolName, args)` - Execute a tool on an MCP server
- `getServer(name)` - Retrieve a running server instance by name
- `getAllServers()` - Get all active MCP server instances
- `stopAllServers()` - Stop all running MCP servers

### MCP Server Configuration

MCP servers are configured in [`data/tools.json`](data/tools.json) with the following structure:

**stdio Transport (Local Servers):**

```json
{
  "name": "memory",
  "type": "mcp",
  "transport": "stdio",
  "command": "npx",
  "args": ["-y", "@modelcontextprotocol/server-memory@latest"],
  "env": {
    "MEMORY_FILE_PATH": "./data/memory.json"
  }
}
```

**http Transport (Hosted Servers):**

```json
{
  "name": "example_server",
  "type": "mcp",
  "transport": "http",
  "url": "https://example.com/mcp-endpoint"
}
```

### Environment Variables

MCP servers can receive environment variables through the `env` field. These are merged with the process environment:

```json
"env": {
  "MEMORY_FILE_PATH": "d:/Projetos/Discord/vica/data/memory.json",
  "CUSTOM_VAR": "value"
}
```

### Memory MCP Server

The Memory MCP server provides knowledge graph operations:

**Available Tools:**
1. `create_entities` - Create multiple entities in the knowledge graph
2. `create_relations` - Create relationships between entities
3. `add_observations` - Add observations to existing entities
4. `delete_entities` - Remove entities and their relations
5. `delete_observations` - Remove specific observations
6. `delete_relations` - Remove relationships from the graph
7. `read_graph` - Read the entire knowledge graph
8. `search_nodes` - Search for nodes based on queries
9. `open_nodes` - Open specific nodes by name

### MCP Server Lifecycle

MCP servers are managed in [`bot.js`](bot.js):

1. **Startup Phase**: All configured MCP servers are initialized when the bot starts
2. **Runtime Phase**: MCP tools are available for AI function calling
3. **Shutdown Phase**: All MCP servers are gracefully stopped on bot exit

### Using MCP Tools

The [`core/tool_loader.js`](core/tool_loader.js) module automatically loads MCP tools and makes them available to the AI:

```javascript
const tool_loader = require('./core/tool_loader');

// Load all tools including MCP tools
const tools = tool_loader.loadTools();

// Use in OpenAI chat completion
const response = await oai.chatCompletion(messages, tools);
```

### Testing MCP Integration

Use the test script to verify MCP server connectivity:

```bash
node test_mcp.js
```

This script:
- Starts all configured MCP servers
- Lists available tools from each server
- Performs a simple tool call test
- Stops all servers gracefully

### Troubleshooting MCP Servers

**HTTP-based servers returning HTML:**

The mcp.so URLs may not be correct MCP HTTP endpoints. Possible issues:
- Different protocol required (SSE, WebSocket)
- Missing authentication headers
- Incorrect endpoint format

**Windows Compatibility:**

- Use `npm.cmd exec` instead of `npx` on Windows
- The [`mcp_client.js`](core/mcp_client.js) automatically adds `shell: true` for Windows

**Server Startup Failures:**

- Verify the command is available in system PATH
- Check environment variables are correctly configured
- Ensure required dependencies are installed
- Review server logs for detailed error messages

### Adding a New MCP Server

1. Add server configuration to [`data/tools.json`](data/tools.json)
2. Select appropriate transport type (stdio or http)
3. Configure required environment variables
4. Test with `node test_mcp.js`
5. Restart the bot to load the new server
6. Verify tools are available for AI function calling

### MCP Best Practices

1. **Prefer stdio transport** for local servers (more reliable and easier to debug)
2. **Use absolute paths** for file-based server configurations
3. **Test thoroughly** before deploying to production
4. **Monitor server health** through logs
5. **Implement fallback logic** when MCP tools fail

### Vibe Check Reflection Policy

When the `vibe_check` tool is available, it must be used for reflection before major actions.

ALWAYS include the full user prompt in every `vibe_check` call to preserve context awareness.

As an autonomous agent:
1. Treat `vibe_check` as a high-priority collaborative pattern interrupt for course correction, not as optional output
2. Use `vibe_check` at strategic points: after planning and before implementation, when complexity increases, and before significant system changes
3. Always provide the current phase in the call (`planning`, `implementation`, or `review`) for context-appropriate feedback
4. Adapt the approach based on `vibe_check` feedback unless it is clearly irrelevant; if ignored, document why
5. Chain `vibe_check` with `vibe_learn` after resolving mistakes to record recurring patterns and successful corrections
6. If `vibe_check` is temporarily unavailable, continue with explicit self-review and log the limitation

### VS Code Symbol Tools for Context Efficiency
Use VS Code symbol tools to reduce context consumption:
- `get_document_symbols_code` for file structure overview instead of reading entire files
- `search_symbols_code` to find symbols by name across the project
- `get_symbol_definition_code` for type info and docs without full file context
- Workflow: get outline → search symbols → get definitions → read implementation only when needed

---

## Database Schema

The bot uses SQLite with the following tables:

### `mensagens`
Stores message history for conversation context.

| Column | Type | Purpose |
|--------|------|---------|
| id | INTEGER PRIMARY KEY | Unique message ID |
| guild_id | TEXT | Discord server ID |
| channel_id | TEXT | Discord channel ID |
| user_id | TEXT | Discord user ID |
| content | TEXT | Message content |
| timestamp | INTEGER | Unix timestamp |

### `rank_xp`
Stores user XP and level information.

| Column | Type | Purpose |
|--------|------|---------|
| guild_id | TEXT | Discord server ID |
| user_id | TEXT | Discord user ID |
| xp | INTEGER | Current XP |
| level | INTEGER | Current level |
| PRIMARY KEY | (guild_id, user_id) | Composite key |

### `blacklist_chatbot_canais`
Channels where the chatbot is disabled.

| Column | Type | Purpose |
|--------|------|---------|
| guild_id | TEXT | Discord server ID |
| channel_id | TEXT | Discord channel ID |
| PRIMARY KEY | (guild_id, channel_id) | Composite key |

### `blacklist_xp_canais`
Channels where XP is not awarded.

| Column | Type | Purpose |
|--------|------|---------|
| guild_id | TEXT | Discord server ID |
| channel_id | TEXT | Discord channel ID |
| PRIMARY KEY | (guild_id, channel_id) | Composite key |

### `rank_role_multipliers`
Role-based XP multipliers.

| Column | Type | Purpose |
|--------|------|---------|
| guild_id | TEXT | Discord server ID |
| role_id | TEXT | Discord role ID |
| multiplier | REAL | XP multiplier (e.g., 1.5 for 50% bonus) |
| PRIMARY KEY | (guild_id, role_id) | Composite key |

### `guild_settings`
Server-specific configuration.

| Column | Type | Purpose |
|--------|------|---------|
| guild_id | TEXT PRIMARY KEY | Discord server ID |
| setting_name | TEXT | Setting identifier |
| setting_value | TEXT | Setting value (JSON string for complex values) |

### `user_memories`
User-specific memories with embeddings.

| Column | Type | Purpose |
|--------|------|---------|
| id | INTEGER PRIMARY KEY | Unique memory ID |
| guild_id | TEXT | Discord server ID |
| user_id | TEXT | Discord user ID |
| content | TEXT | Memory content |
| embedding | TEXT | Embedding vector (JSON array) |
| tags | TEXT | Memory tags (JSON array) |
| timestamp | INTEGER | Unix timestamp |

### `guild_memories`
Server-wide memories with embeddings.

| Column | Type | Purpose |
|--------|------|---------|
| id | INTEGER PRIMARY KEY | Unique memory ID |
| guild_id | TEXT | Discord server ID |
| content | TEXT | Memory content |
| embedding | TEXT | Embedding vector (JSON array) |
| tags | TEXT | Memory tags (JSON array) |
| timestamp | INTEGER | Unix timestamp |

### `role_congrats`
Role congratulations messages.

| Column | Type | Purpose |
|--------|------|---------|
| guild_id | TEXT | Discord server ID |
| role_id | TEXT | Discord role ID |
| message | TEXT | Congratulations message |
| PRIMARY KEY | (guild_id, role_id) | Composite key |

### `reaction_emojis`
Reaction emojis per server.

| Column | Type | Purpose |
|--------|------|---------|
| guild_id | TEXT | Discord server ID |
| emoji | TEXT | Emoji (Unicode or custom) |
| PRIMARY KEY | (guild_id, emoji) | Composite key |

---

## Memory System

### Tag System

The bot uses special tags in messages to trigger memory operations:

| Tag | Purpose | Example |
|-----|---------|---------|
| `[salvar_memoria]` | Save content as a memory | `[salvar_memoria] O usuário gosta de café` |
| `[meta]` | Add metadata to memory | `[meta] preferência: café` |
| `[imagem]` | Include image in memory | `[imagem] (with image attachment)` |

### Embedding System

- Uses OpenAI-compatible API for text embeddings
- Stores embeddings as JSON arrays in database
- Implements cosine similarity for semantic search
- Retrieves relevant memories based on query similarity

### Memory Types

1. **User Memories**: Specific to individual users
   - Stored in `user_memories` table
   - Associated with `user_id`
   - Personal preferences, facts about user

2. **Guild Memories**: Server-wide knowledge
   - Stored in `guild_memories` table
   - Associated with `guild_id`
   - Server rules, common knowledge, shared information

### Memory Retrieval

When processing a message:
1. Generate embedding for the message
2. Search for similar memories using cosine similarity
3. Retrieve top N most relevant memories
4. Include memories in AI context for better responses

---

## Configuration

### config.toml Structure

```toml
[discord]
token = "YOUR_DISCORD_BOT_TOKEN"
client_id = "YOUR_CLIENT_ID"

[models.default]
base_url = "https://your-openai-compatible-api/v1"
api_key = "YOUR_API_KEY"
model = "gpt-4o"

[models.vision]
base_url = "https://your-openai-compatible-api/v1"
api_key = "YOUR_API_KEY"
model = "gpt-4o"

[models.transcriptions]
base_url = "https://your-openai-compatible-api/v1"
api_key = "YOUR_API_KEY"
model = "whisper-1"

[ai_settings]
send_system_prompt = true
retries = 3
initial_delay_ms = 1000
historyLimit = 6
historyMaxAge = 60
budgetTokenLimit = 3000
rate_limit_ms = 5000

[ai_tools]
enabled = true
tools_file = "data/tools.json"
```

Capability-specific sections such as `[models.vision]` and `[models.transcriptions]` can omit fields and inherit missing values from `[models.default]`.

### Deployment

The bot is deployed on Raspberry Pi 3b+ using systemd:

1. **Update code**:
    ```bash
    cd <path/to/vica>
    git pull
    ```

2. **Restart service**:
    ```bash
    systemctl restart vica
    ```

3. **Check status**:
    ```bash
    systemctl status vica
    ```

### Environment Variables

The bot uses `config.toml` for configuration. Ensure sensitive data (API keys, tokens) are properly secured and not committed to version control.

---

## Code Patterns

### File Headers

All source files should include a header with:
- File path
- Creation date
- Author
- Collaboration notes

Example:
```javascript
// core/database.js
// 2024-01-15
// Author: Your Name
// Collaboration: AI Assistant
```

### Logging Pattern

Use consistent logging format: `[MODULE][LEVEL] message`

```javascript
console.log('[DATABASE][INFO] Connected to database');
console.error('[OAI][ERROR] API request failed:', error);
console.warn('[CHATBOT][WARN] Rate limit exceeded for user:', userId);
```

### Database Operations

Always use prepared statements for database queries:

```javascript
const stmt = db.prepare('SELECT * FROM table WHERE id = ?');
const result = stmt.get(id);
```

For multiple operations, use transactions:

```javascript
db.transaction(() => {
  const insert = db.prepare('INSERT INTO table (col) VALUES (?)');
  insert.run(value1);
  insert.run(value2);
})();
```

### Async/Await Patterns

Use async/await for asynchronous operations:

```javascript
async function processMessage(message) {
  try {
    const response = await fetchAPI(message);
    return response;
  } catch (error) {
    console.error('[MODULE][ERROR]', error);
    throw error;
  }
}
```

### Error Handling

Always wrap potentially failing operations in try/catch blocks:

```javascript
try {
  await riskyOperation();
} catch (error) {
  console.error('[MODULE][ERROR]', error);
  // Handle error appropriately
}
```

### Permission Checks

Always check permissions before Discord API calls:

```javascript
if (!message.member.permissions.has(Permissions.FLAGS.ADMINISTRATOR)) {
  return message.reply('You do not have permission to use this command.');
}
```

### Ephemeral Responses

Use ephemeral responses for admin commands to avoid clutter:

```javascript
await interaction.reply({
  content: 'Configuration updated successfully',
  ephemeral: true
});
```

### Component Collectors

Use component collectors with timeouts for interactive elements:

```javascript
const collector = message.createMessageComponentCollector({
  time: 60000 // 60 seconds
});

collector.on('collect', async (i) => {
  await i.update({ content: 'Selected!', components: [] });
});

collector.on('end', (collected, reason) => {
  if (reason === 'time') {
    // Handle timeout
  }
});
```

### Modal-based Input

Use modals for complex configuration input:

```javascript
const modal = new ModalBuilder()
  .setCustomId('configModal')
  .setTitle('Configuration');

const input = new TextInputBuilder()
  .setCustomId('settingInput')
  .setLabel('Setting Value')
  .setStyle(TextInputStyle.Short);

const row = new ActionRowBuilder().addComponents(input);
modal.addComponents(row);

await interaction.showModal(modal);
```

---

## Technical Details

### XP Calculation

XP is awarded randomly within a configurable range:

```javascript
const xp = Math.floor(Math.random() * (max - min + 1)) + min;
```

Default range: 15-25 XP per message

### Rate Limiting

- **Per-user rate limit**: 5 seconds
- Prevents spam and API abuse
- Implemented using timestamp tracking

### Conversation Context

- **Context window**: Last 6 messages
- Includes both user and bot messages
- Maintains conversation flow
- Stored in `mensagens` table

### Message Splitting

Discord has a 2000 character limit per message. The bot splits long responses:

```javascript
const MAX_LENGTH = 2000;
const chunks = [];
for (let i = 0; i < message.length; i += MAX_LENGTH) {
  chunks.push(message.slice(i, i + MAX_LENGTH));
}
```

### Embedding Similarity

Cosine similarity is used for semantic search:

```javascript
function cosineSimilarity(a, b) {
  const dotProduct = a.reduce((sum, val, i) => sum + val * b[i], 0);
  const magnitudeA = Math.sqrt(a.reduce((sum, val) => sum + val * val, 0));
  const magnitudeB = Math.sqrt(b.reduce((sum, val) => sum + val * val, 0));
  return dotProduct / (magnitudeA * magnitudeB);
}
```

### OpenAI Function Calling

Tools are loaded dynamically and passed to the AI:

```javascript
const tools = tool_loader.loadTools();
const response = await openai.chat.completions.create({
  model: 'gpt-4',
  messages: conversation,
  tools: tools
});
```

---

## Development Notes

### Language

- **Primary language**: Portuguese
- All user-facing text should be in Portuguese
- Code comments and variable names can be in English or Portuguese
- Maintain consistency within files

### Error Handling

- Always handle errors gracefully
- Provide user-friendly error messages
- Log errors with context for debugging
- Never expose sensitive information in error messages

### Code Organization

- Keep files focused on single responsibilities
- Use descriptive file and function names
- Follow the existing directory structure
- Maintain separation of concerns (commands, events, core, helpers, tools)

### Testing

- Test commands in a development server first
- Verify database operations don't cause locks
- Check rate limiting works correctly
- Test error scenarios

### Performance Considerations

- Use prepared statements for database queries
- Implement caching where appropriate (auditCache)
- Avoid blocking the event loop
- Use async/await for I/O operations

### Security

- Never commit API keys or tokens
- Use environment variables or config files
- Validate user input
- Check permissions before executing privileged operations
- Sanitize database inputs

### Deployment Checklist

Before deploying:
1. Update `config.toml` with production values
2. Test all critical commands
3. Verify database migrations
4. Check systemd service configuration
5. Monitor logs after deployment

### Common Issues

1. **Database locks**: Ensure transactions are properly committed
2. **Rate limits**: Implement exponential backoff for API calls
3. **Memory leaks**: Clean up event listeners and collectors
4. **Permission errors**: Check bot permissions in Discord server

---

## File Structure

```
vica/
├── bot.js                          # Main bot entry point
├── config.example.toml             # Example configuration
├── delete-commands.js              # Delete slash commands
├── deploy-commands.js              # Deploy slash commands
├── package.json                    # Node.js dependencies
├── test_mcp.js                     # MCP integration test script
├── README.MD                       # Project documentation
├── GEMINI.MD                       # This file
│
├── commands/                       # Slash commands
│   ├── canal_noticia.js
│   ├── chat_blacklist.js
│   ├── comment.js
│   ├── config.js
│   ├── deslurkar.js
│   ├── enex.js
│   ├── gmemories.js
│   ├── mmemories.js
│   ├── noticia.js
│   ├── perguntar.js
│   ├── rank.js
│   ├── rank_blacklist.js
│   ├── rank_reset.js
│   ├── rank_rolexp.js
│   ├── rank_setxp.js
│   ├── reaction_emoji.js
│   ├── reaction_thread.js
│   ├── system_channel.js
│   ├── trigger.js
│   └── up_role.js
│
├── core/                           # Core functionality
│   ├── auditCache.js
│   ├── database.js
│   ├── mcp_client.js               # MCP client for external tools
│   ├── oai_interface.js
│   ├── static_data.js
│   ├── tagParser.js
│   ├── tool_loader.js              # Loads tools and MCP servers
│   └── transcriber.js
│
├── events/                         # Discord event handlers
│   ├── guildBanAdd.js
│   ├── guildMemberAdd.js
│   ├── guildMemberRemove.js
│   ├── guildMemberUpdate.js
│   ├── interactionCreate.js
│   ├── messageCreate.js
│   └── messageReactionAdd.js
│
├── helpers/                        # Helper utilities
│   └── embeddingHelper.js
│
└── tools/                          # OpenAI function tools
    ├── calculate.js
    └── get_current_time.js
```

---

## Additional Resources

- [Discord.js Documentation](https://discord.js.org/)
- [OpenAI API Documentation](https://platform.openai.com/docs/)
- [Node.js Documentation](https://nodejs.org/docs/)
- [SQLite Documentation](https://www.sqlite.org/docs.html)

---

## Contributing

When contributing to this project:

1. Follow the existing code patterns and style
2. Add appropriate logging with the `[MODULE][LEVEL]` format
3. Include file headers with path, date, author, and collaboration notes
4. Test changes thoroughly before committing
5. Update this documentation if adding new features or changing architecture

---

## License

See [`LICENSE`](LICENSE) file for details.

---

*Last updated: 2026-03-09*
*Document version: 1.2*

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/vico93)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/vico93)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
