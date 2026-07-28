## devpilot-intellij

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

DevPilot is an IntelliJ IDEA plugin that serves as an AI coding assistant. It integrates various AI capabilities into JetBrains IDEs, including code suggestions, bug detection, code refactoring, test generation, and code explanation. The plugin has recently added integration with Claude Code via terminal.

## Build and Development Commands

### Building and Running the Plugin

```bash
# Build and run the plugin (launches a development instance of IntelliJ)
./gradlew runIde

# On Windows:
gradlew.bat runIde
```

### Testing and Verification

```bash
# Run all tests and verification tasks
./gradlew check

# Compile Java files
./gradlew compileJava

# Configure plugin metadata
./gradlew patchPluginXml

# Code style verification
./gradlew checkstyle
```

### Plugin Deployment (for maintainers)

```bash
# Sign the plugin for distribution (requires credentials)
./gradlew signPlugin

# Publish to JetBrains Marketplace (requires token)
./gradlew publishPlugin
```

## Project Architecture

### Key Components

1. **Plugin Core**:
   - Entry point through `plugin.xml` with extensions for tool windows and actions
   - Main UI provided via custom tool window (`DevPilotChatToolWindowFactory`)
   - Settings management with persistent state components

2. **UI Components**:
   - Web-based chat interface using JCEF (Java Chromium Embedded Framework)
   - Custom terminal integration for Claude Code
   - Toolbar actions for quick access
   - Status bar widgets for state indication

3. **AI Integration**:
   - MCP (Model Control Protocol) for model server communication
   - Chat session management with history preservation
   - Code operations (insertion, replacement, new file creation)
   - Terminal-based Claude Code integration

4. **Terminal Integration**:
   - Custom terminal handling for Claude Code with ESC key capture
   - Environment variable management for local proxy configuration
   - Terminal session reuse and management

### Important Packages

- `com.zhongan.devpilot.actions`: Editor and toolbar actions
- `com.zhongan.devpilot.cli`: Terminal and CLI utilities
- `com.zhongan.devpilot.completions`: Code completion functionality
- `com.zhongan.devpilot.gui`: UI components and toolwindows
- `com.zhongan.devpilot.mcp`: MCP server management
- `com.zhongan.devpilot.settings`: Configuration and settings
- `com.zhongan.devpilot.webview`: Web view handling

### Configuration Files

- `.mcp.json`: Project-level MCP server configuration (for Claude Code)
- `mcp_configuration.json`: User-level MCP configuration (settings dir)
- `plugin.xml`: Core plugin definition and extension points

## Development Notes

### Claude Code Integration

The recent development focus has been on adding Claude Code integration to the DevPilot plugin. This integration allows:

1. Launching Claude Code from the DevPilot toolbar
2. Handling Claude Code in IntelliJ's terminal with improved ESC key handling
3. Persisting MCP server configurations to project files

When working with the Claude Code integration:
- Check `ToolbarClaudeCodeAction.java` for the toolbar button implementation
- See `TerminalUtil.java` for terminal handling and command execution
- Review `McpConfigurationHandler.java` for configuration management

### Dependencies

The plugin depends on:
- IntelliJ Platform SDK (targeting 2022.3 to 2025.2.*)
- Java 17
- Various IntelliJ bundled plugins (Java, Git4Idea, Terminal)
- External libraries for HTTP communication and JSON processing

### WebView Integration

For updates to the WebView component:
1. Changes should be made in the separate `devpilot-h5` repository
2. Built using `pnpm install` and `pnpm run build`
3. HTML file copied to `src/main/resources/webview/index.html`

### Important Configurations

- The plugin requires Java 17 or higher
- It supports IntelliJ IDEs from version 2022.3 (223) to 2025.2.* (252)
- For complete functionality, it requires external services:
  - AI gateway (can be configured in settings)
  - Authentication system (can be disabled)
  - Telemetry system (can be disabled)

---
> Source: [openpilot-hub/devpilot-intellij](https://github.com/openpilot-hub/devpilot-intellij) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-23 -->
