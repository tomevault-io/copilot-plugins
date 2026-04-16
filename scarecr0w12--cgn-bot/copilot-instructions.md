## cgn-bot

> Multi-provider AI system architecture, rate limiting, usage tracking, and conversation management


# AI Integration System

The AI subsystem provides multi-provider conversation management with vector memory, usage tracking, and rate limiting.

**Importance Score: 95/100**

## Architecture Overview

```text
AIManager.js (18KB - Main Orchestrator)
       │
       ├── providers/
       │   ├── ProviderFactory.js
       │   ├── OpenAIProvider.js
       │   ├── AnthropicProvider.js
       │   ├── GroqProvider.js
       │   ├── OllamaProvider.js
       │   └── OpenAICompatibleProvider.js
       │
       ├── ConversationMemory.js
       ├── VectorMemory.js (12KB)
       ├── RateLimiter.js
       └── UsageTracker.js (6.6KB)
```

Path: `Modules/AI/`

## Supported Providers

| Provider | File | Use Case |
|----------|------|----------|
| OpenAI | `OpenAIProvider.js` | GPT-4, GPT-3.5 |
| Anthropic | `AnthropicProvider.js` | Claude models |
| Groq | `GroqProvider.js` | Fast inference |
| Ollama | `OllamaProvider.js` | Local LLM server |
| OpenAI-Compatible | `OpenAICompatibleProvider.js` | LocalAI, vLLM, Text Gen WebUI |

## Key Components

### AIManager.js

Main orchestrator (18KB) handling:

- Provider selection and fallback
- Conversation context management
- Response generation
- Error handling and retries

### ConversationMemory.js

Chat history management:

- Per-channel conversation storage
- Context window management
- Message pruning for token limits

### VectorMemory.js

Semantic search for contextual conversation history:

- Vector embeddings for messages
- Similarity search for relevant context
- 12KB implementation

### UsageTracker.js

Budget and quota enforcement (6.6KB):

- Per-guild usage tracking
- Per-user usage tracking
- Budget limits and warnings
- Usage statistics and reporting

### RateLimiter.js

Provider-specific rate limiting:

- Request rate tracking
- Cooldown enforcement
- Provider quota management

## Tool System

Path: `Modules/AI/tools/`

| Tool | Purpose |
|------|---------|
| `ToolRegistry.js` | Tool registration and dispatch |
| `WebSearchTool.js` | Web search integration |

## Dashboard Integration

### Dynamic Model List Loading

Route: `GET /:svrid/ai/models`

- Guarded by `checkUnavailableAPI` and `authorizeDashboardAccess` middleware
- Returns available models from configured providers
- Server-specific model availability

### Server AI Settings

Schema: `Database/Schemas/serverAISchema.js` (4.7KB)

Configurable per-server:

- Enabled/disabled status
- Default provider selection
- Model preferences
- Usage limits
- System prompt customization

## Premium Gating

AI features are gated by the tier system:

```javascript
const TierManager = require('../../Modules/TierManager');

// Premium is per-server - use guild ID
const hasAccess = await TierManager.canAccess(msg.guild.id, 'ai_chat');
if (!hasAccess) {
    return msg.reply('This feature requires a premium subscription for this server.');
}
```

Features:

- `ai_chat` - Basic AI conversation
- `ai_images` - AI image generation (DALL-E, Stable Diffusion)

## Environment Configuration

```bash
# OpenAI
OPENAI_API_KEY=sk-...

# Anthropic
ANTHROPIC_API_KEY=sk-ant-...

# Groq
GROQ_API_KEY=gsk_...

# Ollama (local)
OLLAMA_BASE_URL=http://localhost:11434

# OpenAI-Compatible
OPENAI_COMPATIBLE_BASE_URL=http://localhost:8080
OPENAI_COMPATIBLE_API_KEY=...
```

## Usage Patterns

### Basic Conversation

```javascript
const AIManager = require('./Modules/AI/AIManager');

const response = await AIManager.generateResponse({
    guildId: guild.id,
    userId: user.id,
    channelId: channel.id,
    message: userMessage,
    context: previousMessages
});
```

### With Provider Selection

```javascript
const response = await AIManager.generateResponse({
    guildId: guild.id,
    userId: user.id,
    provider: 'anthropic',
    model: 'claude-3-sonnet',
    message: userMessage
});
```

## Key Files

| File | Size | Purpose |
|------|------|---------|
| `AIManager.js` | 18KB | Main orchestrator |
| `VectorMemory.js` | 12KB | Semantic search |
| `UsageTracker.js` | 6.6KB | Budget tracking |
| `providers/` | - | 7 provider implementations |
| `tools/` | - | Tool system (2 tools) |

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/scarecr0w12) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
