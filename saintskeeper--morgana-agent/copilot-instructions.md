## morgana-agent

> This file provides guidance to Claude Code (claude.ai/code) when working with

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with
code in this repository.

## Project Overview

Morgana Agent is a Go-based AI agent REPL that interfaces with Ollama for local
LLM interactions. The agent includes file search capabilities, tool calling
support, and an interactive command-line interface.

## Development Commands

### Build and Run

```bash
# Run the agent directly
go run main.go

# Build the binary
go build -o morgana-agent

# Run with environment variables
cp .env.example .env  # Configure your settings
go run main.go
```

### Dependencies

```bash
# Install dependencies
go mod download

# Update dependencies
go mod tidy
```

## Architecture

### Core Components

1. **Main Agent (`main.go`)**

   - REPL interface for interacting with Ollama models
   - Tool calling system with automatic detection and execution
   - Context management and conversation history
   - Command system (prefix with `/` for commands)

2. **File Search Module (`internal/search/`)**

   - Pattern-based file searching with gitignore support
   - Content grep functionality
   - Project structure visualization (tree view)
   - Configurable search options with filters

3. **Command Registry System (`internal/repl/command_registry.go`)**

   - Centralized metadata system for all REPL commands
   - Thread-safe command registration and lookup
   - Command categorization (system, model, search, tools, appearance)
   - Parameter definition with validation metadata
   - Fuzzy search and autocomplete support
   - Structured help text generation
   - Extensible architecture for future commands

4. **Tools System (`internal/tools/`)**

   - Tool registry for extensible functionality
   - Built-in tools:
     - `search_files`: Pattern-based file search
     - `find_in_files`: Content grep functionality
     - `show_tree`: Project structure visualization
     - `read_file`: Read file contents
     - `fuzzy_find`: Fuzzy file/directory search
     - `write_file`: Create or write files with atomic operations
     - `edit_file`: Edit files with replace/insert/delete operations
     - `set_approval_mode`: Change file operation approval mode
     - `show_pending_changes`: View pending file changes
     - `apply_pending_changes`: Apply approved changes
   - Tool call parsing from LLM responses
   - Parameter validation and execution
   - Security boundaries for file operations
   - **Approval System**: Interactive file change preview and confirmation
     - Shows diffs and previews before applying changes
     - Four modes: interactive, auto, confirm, dry-run
     - Color-coded change visualization
     - Smart environment detection and mode compatibility

5. **Enhanced Prompt System (`internal/repl/prompt_system.go`)**
   - Model-specific prompt templates for improved tool usage
   - Progressive prompting with retry logic and failure recovery
   - Dynamic prompt building based on model capabilities
   - Approval mode awareness with context injection
   - Performance metrics tracking and persistence
   - Template hot-reloading for development
   - Model profiles with capability detection:
     - **Qwen**: Structured format emphasis, reasoning-based approach
     - **Llama**: Tool-focused with context awareness
     - **GPT**: Excellence in tool usage with efficiency focus
     - **Claude**: Analytical approach with thoughtful explanations
     - **Gemma**: Example-driven with step-by-step guidance

### Environment Configuration

The agent uses environment variables (via `.env` file):

- `OLLAMA_HOST`: Ollama server endpoint (default: `http://localhost:11434`)
- `OLLAMA_MODEL`: Model to use (default: `llama3.2`)
- `AGENT_NAME`: Agent display name (default: `Morgana`)
- `AGENT_TEMPERATURE`: Response temperature (default: `0.7`)
- `AGENT_MAX_TOKENS`: Maximum response tokens (default: `2048`)
- `AGENT_ENABLE_TOOLS`: Enable/disable tool calling (default: `true`)
- `AGENT_APPROVAL_MODE`: File operations approval mode (default: smart
  detection)
  - `interactive`: Shows preview and asks for confirmation
  - `auto`: Automatically applies all file changes
  - `confirm`: Batch preview with single yes/no decision
  - `dry-run`: Shows preview but never applies changes
- `AGENT_THEME_MODE`: Color theme mode (default: `auto`)
  - `auto`: Adaptive colors based on terminal detection
  - `light`: Forced light color scheme
  - `dark`: Forced dark color scheme
- `MORGANA_BUBBLE_TEA`: Force Bubble Tea UI mode (default: auto-detect)

### Key Design Patterns

- **Tool Registry Pattern**: Extensible tool system with self-describing
  parameters
- **Search Options**: Flexible configuration for file/content searches
- **REPL Commands**: Slash-command system for agent control
- **Context Preservation**: Maintains conversation context across interactions
- **Streaming Responses**: Real-time output from Ollama API
- **Atomic File Operations**: Safe file writes using temp file + rename pattern
- **Security Sandboxing**: Path validation to prevent writes outside project

## REPL Commands

- `/help` - Show available commands
- `/clear` - Clear conversation context
- `/history` - Show conversation history
- `/model` - Show current model
- `/switch <model>` - Switch to different model
- `/search <pattern>` - Search files by name pattern
- `/find <text>` - Find text in files
- `/fuzzy <query>` - Fuzzy search for files
- `/tree [depth]` - Show project structure
- `/tools on/off/list` - Manage tool calling
- `/approval [mode]` - Set file approval mode (interactive/auto/confirm/dry-run)
- `/config <subcommand>` - Configuration management (show/validate/print/reload)
- `/prompt [cmd]` - Enhanced prompt system (status/metrics/profile/reload)
- `/theme [mode]` - Color theme management (auto/light/dark)
- `/exit` or `/quit` - Exit REPL

## Enhanced Features & Fixes

### Tool Exposure Enhancement ✅ RESOLVED

**Previous Issue**: Morgana wasn't using file creation/editing tools
effectively. **Solution**: Implemented enhanced prompt system with
model-specific templates and progressive prompting. The system now:

- Provides model-specific tool usage guidance
- Includes all available tools in prompts with proper examples
- Adapts prompt complexity based on model capabilities and retry attempts
- Tracks success/failure metrics to improve future prompts

### Color Palette System

The theme system (`internal/theme/`) provides adaptive colors:

- **Auto Mode**: Detects terminal capabilities and adjusts accordingly
- **Light Mode**: Optimized for light backgrounds with dark text
- **Dark Mode**: Optimized for dark backgrounds with light text
- **Color Components**: Primary, secondary, accent, success, warning, error
- **UI Elements**: Borders, highlights, backgrounds, text colors

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/saintskeeper)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/saintskeeper)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
