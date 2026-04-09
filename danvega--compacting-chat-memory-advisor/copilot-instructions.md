## compacting-chat-memory-advisor

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Spring Boot 3.5.6 application demonstrating a custom Spring AI advisor that compacts conversation history by summarizing old messages instead of dropping them.

**Dual-Model Setup**: Uses OpenAI GPT-5 for primary chat responses and Google Gemini 2.5 Flash for cost-effective summarization. Uses Java 25 and Maven for build management.

## Core Components

### CompactingChatMemoryAdvisor
**Location**: `src/main/java/dev/danvega/compact/advisor/CompactingChatMemoryAdvisor.java`

Custom `CallAdvisor` implementation that manages conversation memory intelligently:
- **Automatically summarizes** oldest messages when threshold is reached
- **Preserves context** from earlier in the conversation (unlike standard memory which drops messages)
- **Reduces token usage** by replacing N messages with a single summary
- **Configurable** via application.properties

**Key Methods**:
- `adviseCall()` - Main advisor logic, intercepts chat requests to manage memory
- `compact()` - Manually trigger compaction
- `clear()` - Clear conversation history

**Integration**: Implements Spring AI's `CallAdvisor` interface, used with `ChatClient.Builder`

### Configuration
**Location**: `src/main/java/dev/danvega/compact/config/`

- `ChatMemoryConfiguration.java` - Bean definitions for regular and compacting memory
- `CompactingMemoryProperties.java` - Externalized configuration properties

### Controllers
**Location**: `src/main/java/dev/danvega/compact/`

- `ChatMemoryController.java` - Standard memory endpoint (for comparison)
- `CompactingChatMemoryController.java` - Compacting memory endpoint with clear/trigger operations

## Development Commands

### Build
```bash
./mvnw clean install
```

### Run Application
```bash
./mvnw spring-boot:run
```

### Run Tests
```bash
./mvnw test
```

### Run Single Test
```bash
./mvnw test -Dtest=ClassName#methodName
```

## Configuration

### Required Environment Variables
- `OPENAI_API_KEY`: OpenAI API key for primary chat model (GPT-5)
- `GOOGLE_GENAI_API_KEY`: Google GenAI API key for summarization model (Gemini 2.5 Flash)

### Spring AI Configuration
Located in `src/main/resources/application.properties`:

**Primary Chat Model (OpenAI GPT-5)**:
- Model: gpt-5
- Temperature: 1.0
- API key sourced from `OPENAI_API_KEY` environment variable

**Summarization Model (Google Gemini 2.5 Flash)**:
- Model: gemini-2.5-flash
- API key sourced from `GOOGLE_GENAI_API_KEY` environment variable
- Used exclusively for compacting/summarizing old messages (cost optimization)

### Compacting Memory Configuration
Located in `src/main/resources/application.properties`:
- `compact.memory.max-messages` - Maximum messages to retain (default: 100)
- `compact.memory.compact-threshold` - When to trigger auto-compaction (default: 80)
- `compact.memory.messages-to-compact` - How many old messages to summarize (default: 40)

## Architecture

### Spring AI Advisor Pattern
This project demonstrates implementing a custom `CallAdvisor` that intercepts chat requests:

1. **User sends message** → `CompactingChatMemoryController`
2. **Controller uses ChatClient** with `CompactingChatMemoryAdvisor`
3. **Advisor checks** if compaction needed (message count ≥ threshold)
4. **If threshold reached**: Summarize oldest messages via **Gemini 2.5 Flash** (cost-effective)
5. **Advisor adds** user message to memory
6. **Advisor augments** prompt with full conversation history
7. **ChatModel (OpenAI GPT-5) processes** request with conversation context
8. **Advisor stores** assistant response in memory

### Dual-Model Architecture
- **OpenAI GPT-5**: Handles all user-facing chat responses (high quality)
- **Google Gemini 2.5 Flash**: Handles background summarization tasks (cost-effective)
- Configuration in `ChatMemoryConfiguration.java` uses `@Qualifier` to inject the appropriate model

### Key Classes
- **Main application**: `dev.danvega.compact.Application` - Spring Boot entry point
- **Advisor**: `CompactingChatMemoryAdvisor` - Custom memory management logic
- **Config**: `ChatMemoryConfiguration` - Bean wiring for memory and advisor
- **Properties**: `CompactingMemoryProperties` - Externalized config values
- **Dependencies**: Spring Web for REST endpoints, Spring AI OpenAI starter for AI model integration

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/danvega)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/danvega)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
