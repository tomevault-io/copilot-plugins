## animal-grooming

> AI CLI MCP Server tool documentation following the SuperClaude framework format.

# AI-CLI MCP Tools Documentation

AI CLI MCP Server tool documentation following the SuperClaude framework format.

## Tool System Architecture

### Core Tool Structure
```yaml
---
tool: "tool-name"
category: "Primary classification"
purpose: "Operational objective"
thread-aware: true|false
performance-profile: "real-time|standard|batch"
---
```

### Tool Processing Pipeline
1. **Input Validation**: Parameter validation and type checking
2. **Thread Management**: Thread ID resolution and persistence
3. **Model Selection**: AI model routing and validation
4. **Request Execution**: Stream processing and response handling
5. **State Management**: Thread state persistence across calls

### Integration Layers
- **MCP Protocol**: Standard Model Context Protocol implementation
- **Thread System**: Persistent conversation state management
- **Model Router**: Multi-provider AI model integration
- **Stream Handler**: Real-time response streaming

## Thread Management System

**Thread Persistence Engine**: File-based state management for conversation continuity. Thread IDs persist across MCP tool calls with 24-hour auto-expiration.

**Thread-Aware Tools**:
- **Primary**: `new-chat`, `continue-chat`, `ai-chat`
- **Management**: `create-thread`, `get-current-thread`, `clear-thread`

### Conversation Tools

**`ai-chat [model] [message] [threadId?] [newThread?]`**
```yaml
---
tool: "ai-chat"
category: "Conversation & AI Interaction"
purpose: "Flexible AI model communication with thread control"
thread-aware: true
performance-profile: "real-time"
---
```
- **Parameters**: 
  - `model`: Target AI model (required)
  - `message`: User message (required)
  - `threadId`: Specific thread ID (optional)
  - `newThread`: Force new thread creation (optional)
- **Thread Behavior**: Uses provided threadId, creates new if absent or newThread=true
- **Response Format**: `[Thread: ID]\n\nAI Response`

**`new-chat [model] [message]`**
```yaml
---
tool: "new-chat"
category: "Conversation & AI Interaction"
purpose: "Start fresh conversation with new thread"
thread-aware: true
performance-profile: "real-time"
---
```
- **Parameters**: 
  - `model`: Target AI model (required)
  - `message`: Initial message (required)
- **Thread Behavior**: Always creates new thread
- **Use Case**: New topics, context reset

**`continue-chat [model] [message]`**
```yaml
---
tool: "continue-chat"
category: "Conversation & AI Interaction"
purpose: "Continue existing conversation thread"
thread-aware: true
performance-profile: "real-time"
---
```
- **Parameters**: 
  - `model`: Target AI model (required)
  - `message`: Continuation message (required)
- **Thread Behavior**: Uses current thread, creates if none exists
- **Use Case**: Ongoing conversations, follow-up questions

### Thread Management Tools

**`create-thread`**
```yaml
---
tool: "create-thread"
category: "Thread Management"
purpose: "Create conversation thread without message"
thread-aware: true
performance-profile: "standard"
---
```
- **Parameters**: None
- **Returns**: New thread ID
- **Side Effect**: Sets as current thread

**`get-current-thread`**
```yaml
---
tool: "get-current-thread"
category: "Thread Management"
purpose: "Retrieve active thread identifier"
thread-aware: true
performance-profile: "standard"
---
```
- **Parameters**: None
- **Returns**: Current thread ID or null message
- **Use Case**: Thread verification, debugging

**`clear-thread`**
```yaml
---
tool: "clear-thread"
category: "Thread Management"
purpose: "Reset thread state for fresh start"
thread-aware: true
performance-profile: "standard"
---
```
- **Parameters**: None
- **Side Effect**: Clears current thread
- **Use Case**: Context reset, new sessions

### Model Discovery Tools

**`list-models`**
```yaml
---
tool: "list-models"
category: "Model Discovery"
purpose: "Enumerate available AI models"
thread-aware: false
performance-profile: "standard"
---
```
- **Parameters**: None
- **Returns**: Formatted list of 17 available models
- **Model Categories**: Anthropic, OpenAI, Google, Perplexity, Others

## Supported AI Models

### Model Categories
- **Anthropic Claude**: `anthropic_claude-3-haiku`, `anthropic_claude-3.7-sonnet`
- **OpenAI GPT**: `openai_gpt-4`, `openai_gpt-4.1`, `openai_gpt-4.1-mini`, `openai_gpt-4.1-nano`, `openai_o3`, `openai_o4-mini`
- **Google Gemini**: `google_gemini`, `google_gemini-2.0-flash`
- **Perplexity**: `perplexity_sonar-pro`, `perplexity_sonar-large-online`, `perplexity_sonar-reasoning`, `perplexity_sonar-reasoning-pro`, `perplexity_r1`
- **Specialized**: `liner_liner-pro`, `ax_llm-ctl-adot`

## Tool Execution Matrix

### Performance Profiles
```yaml
real-time: "Stream processing with immediate response"
standard: "Synchronous execution with moderate latency"
batch: "Asynchronous processing for bulk operations"
```

### Tool Categories
- **Conversation**: new-chat, continue-chat, ai-chat
- **Thread Management**: create-thread, get-current-thread, clear-thread
- **Model Discovery**: list-models

### Thread-Aware Tools
All conversation tools: `ai-chat`, `new-chat`, `continue-chat`
Management tools: `create-thread`, `get-current-thread`, `clear-thread`

## Usage Patterns

### Basic Conversation Flow
```bash
# 1. First Start new conversation
Tool: new-chat
Parameters: { model: "anthropic_claude-3-haiku", message: "Hello!" }

# 2. Continue conversation
Tool: continue-chat
Parameters: { model: "anthropic_claude-3-haiku", message: "Tell me more" }

# 3. New topic (new thread)
Tool: new-chat
Parameters: { model: "openai_gpt-4", message: "Different topic" }
```

### Advanced Thread Control
```bash
# Check current thread
Tool: get-current-thread

# Specific thread conversation
Tool: ai-chat
Parameters: { 
  model: "google_gemini",
  message: "Continue previous discussion",
  threadId: "specific-thread-id"
}

# Force new thread
Tool: ai-chat
Parameters: {
  model: "perplexity_sonar-pro",
  message: "Start fresh",
  newThread: true
}
```

### Session Management
```bash
# Clear session
Tool: clear-thread

# Next conversation creates new thread
Tool: continue-chat
Parameters: { model: "anthropic_claude-3-haiku", message: "Fresh start" }
```

## Technical Implementation

### Thread Persistence
- **Storage**: `/tmp/ai-cli-current-thread.json`
- **Format**: `{ currentThreadId: string, lastUpdated: timestamp }`
- **Expiration**: 24-hour automatic cleanup
- **Error Handling**: Graceful fallback on read/write failures

### Environment Configuration
- **Required**: `AI_CLI_AUTH_TOKEN` environment variable
- **MCP Config**: Standard stdio transport
- **Node Version**: ≥18.19.0

### API Integration
- **Endpoint**: `https://ext.a2a.apollo-ai.io/v1/stream/chat`
- **Protocol**: HTTPS with SSE streaming
- **Authentication**: Bearer token from environment

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/elice365)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/elice365)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
