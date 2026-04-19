## kokibot

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Kokibot is an AI assistant framework built with Kotlin and Spring Boot. It provides a pluggable architecture for LLM providers, communication channels (Telegram), and tools that extend the assistant's capabilities. The assistant uses iterative reasoning with tool calls to answer user queries.

## Development Commands

### Build & Test
```bash
# Full build with tests
mvn clean install

# Run tests only
mvn test

# Run a specific test class
mvn test -Dtest=AssistantTest

# Run a specific test method
mvn test -Dtest=AssistantTest#process

# Run the application
mvn spring-boot:run
```

### Code Quality
```bash
# Lint Kotlin code
mvn antrun:run@ktlint

# Auto-format Kotlin code
mvn antrun:run@ktlint-format

# View JaCoCo coverage report (after running tests)
open target/site/jacoco/index.html
```

### Code Coverage Requirements
- **Line coverage:** 98% minimum (jacoco.threshold.line)
- **Class coverage:** 95% minimum (jacoco.threshold.class)
- Application.java is excluded from coverage requirements

## Architecture

### Core Components

**Assistant** (`Assistant.kt`)
- Main agent that processes user prompts through iterative LLM calls
- Implements a reasoning loop with max 10 iterations (configurable)
- Manages tool execution and memory across iterations
- Loads system instructions from `AGENT.md` in home directory
- Maintains conversation context through ChatHistory

**Bootstrap** (`Bootstrap.kt`)
- Spring service that initializes the entire system
- Sets up LLM provider, channels, and tools from configuration
- Home directory: `~/kokibot` (configurable via `user.home` property)
- Manages lifecycle with @PostConstruct and @PreDestroy hooks

**Context** (`Context.kt`)
- Immutable context object passed to all components
- Contains: home directory, LLM instance, ToolRegistry, ChatHistory

### LLM Integration

- Factory pattern for pluggable LLM providers (`LLMFactory`)
- Currently supported: Deepseek
- All LLMs implement the `LLM` interface with `completion(request)` method
- LLMs are initialized with configuration map and ToolRegistry for function calling

### Tools System

Tools extend the assistant's capabilities (web search, date/time, etc).

**Tool Interface:**
- `init(config, context)` - Initialize with configuration
- `metadata()` - Return tool name, description, parameters
- `exec(arguments)` - Execute the tool with parsed arguments
- `destroy()` - Cleanup resources

**Adding a New Tool:**
1. Create class implementing `Tool` interface in `tools/` package
2. Add to `discoverTools()` list in `Bootstrap.kt`
3. Optionally add config in `~/kokibot/tools/{tool-name}.json`
4. Tool is automatically registered and made available to LLM

**Existing Tools:**
- `TodayTool` - Get current date
- `TimeTool` - Get current time
- `WebSearchTool` - Search the web via Brave Search API
- `WebFetchTool` - Fetch and extract content from URLs (supports HTML, PDF)

### Channels System

Channels are communication interfaces for the assistant.

- Factory pattern for pluggable channels (`ChannelFactory`)
- Currently supported: Telegram
- All channels extend `Channel(agent)` abstract class
- Channels delegate message processing to the Assistant

**Adding a New Channel:**
1. Create class extending `Channel` in `channel/` package
2. Add to `ChannelFactory.create()` method
3. Add configuration in `~/kokibot/settings.json` channels array

### Configuration

**Main Configuration** (`~/kokibot/settings.json`)
```json
{
  "max-iterations": 10,
  "llm": {
    "type": "deepseek",
    "api-key": "${DEEPSEEK_API_KEY}",
    "model": "deepseek-chat"
  },
  "channels": [
    {
      "type": "telegram",
      "token": "${TELEGRAM_TOKEN}"
    }
  ]
}
```

- Environment variables are substituted via `${VAR_NAME}` syntax
- System instructions loaded from `~/kokibot/AGENT.md` (if exists)
- Per-tool configuration in `~/kokibot/tools/{tool-name}.json`

### Memory & Conversation

**ChatHistory** (`memory/ChatHistory.kt`)
- Persists conversation history to `~/kokibot/history.json`
- Loaded into each prompt under "# Conversation history"
- Assistant maintains per-iteration memory for reasoning steps

**Prompt Structure:**
1. User query
2. Previous reasoning steps and tool observations (if any)
3. Full conversation history in JSON format

## Testing

- Framework: JUnit 5
- Mocking: mockito-kotlin (`com.nhaarman.mockitokotlin2`)
- Tests use `mock<Type>()` and `whenever(...).doReturn(...)` pattern
- Test resources in `src/test/resources/` (referenced via `getResourceFile()` helper)
- Spring Boot test utilities available but most tests are pure unit tests

## Code Style

- 4-space indentation (2 for JSON/YAML/XML)
- K&R brace style
- ktlint_official code style with many rules disabled (see .editorconfig)
- No max line length enforcement
- Insert final newline, trim trailing whitespace

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/wutsi) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
