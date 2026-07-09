## dify-a2a-plugin

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Dify plugin that implements the Agent-to-Agent (A2A) protocol, allowing Dify agents to communicate with and delegate tasks to external agents. The plugin provides tools for discovering, calling, and managing remote agents through JSON-RPC 2.0.

## Development Commands

### Plugin Development and Testing

```bash
# Install Python dependencies
# Note: Manifest requires Python 3.12, but newer versions may work in development
python3 -m venv .venv
source .venv/bin/activate  # On Mac/Linux
pip install -r requirements.txt

# Run unit tests
python3 -m unittest tools/tests/test_a2a_registry.py

# Build the plugin package (creates .difypkg file)
dify plugin package

# Install the Dify CLI (if not already installed)
brew tap langgenius/dify
brew install dify
```

**Important Notes:**
- Manifest specifies Python 3.12 as runtime requirement
- Local development may work with newer Python versions (e.g., 3.14)
- Production deployment in Dify will use Python 3.12 as specified in manifest

### Local Testing with Dify Plugin Runtime

For testing with the actual Dify plugin runtime environment:

1. **Using Dify CLI debug mode:**
   ```bash
   dify plugin start --debug
   ```

2. **With dify-plugin-daemon** (if available locally):
   - `dify-plugin-daemon` is not included in this repository (gitignored)
   - It's a local reference copy of the Dify plugin runtime
   - See Dify documentation for setting up local daemon

## Architecture Overview

### Plugin Structure

This is a **Tool Provider Plugin** that exposes 5 tools for A2A protocol operations:

1. **list_agents** - Lists all available agents from the registry
2. **get_agent_capabilities** - Fetches agent card from `/.well-known/agent-card.json` (fallback to `agent.json`)
3. **call_agent** - Synchronous agent invocation via `message/send` JSON-RPC method
4. **submit_task** - Asynchronous task submission via `message/stream` with SSE (extracts taskId from first event)
5. **get_task_status** - Check status of async tasks via `tasks/get` JSON-RPC method

### Key Components

```
├── manifest.yaml              # Plugin metadata and configuration
├── main.py                    # Plugin entrypoint (initializes DifyPluginEnv)
├── provider/
│   ├── a2a.yaml              # Provider definition and credentials schema
│   └── a2a.py                # A2AProvider class with credential validation
├── tools/
│   ├── list_agents.py/yaml   # Tool implementations and definitions
│   ├── get_agent_capabilities.py/yaml
│   ├── call_agent.py/yaml
│   ├── submit_task.py/yaml
│   ├── get_task_status.py/yaml
│   └── tests/                # Unit tests with mocked dependencies
├── _assets/                   # Icon/logo assets
│   ├── a2a-logo-black.svg    # Plugin icon (black version with white background)
│   └── a2a-logo-white.svg    # Plugin icon (white version)
├── screenshots/               # Documentation screenshots for README (18 PNG files)
├── LICENSE                    # Apache 2.0 License
├── PRIVACY.md                 # Privacy policy
└── README.md                  # User-facing documentation
```

### Agent Registry Configuration

The plugin uses **individual credential fields** for up to 5 agents, with each agent having:
- `agent_N_name` - Agent identifier (text-input)
- `agent_N_url` - Base URL endpoint (text-input)
- `agent_N_auth_type` - Authentication type (select: none/bearer/api-key/basic)
- `agent_N_api_key` - API key or token (secret-input)
- `agent_N_description` - Agent description (text-input)

Where N = 1 through 5.

This registry is:
- Defined as individual fields in `provider/a2a.yaml`
- Validated in `provider/a2a.py:_validate_credentials()`
- Built at runtime by each tool via `_build_agents_registry()` method
- Accessed from `self.runtime.credentials` in tool implementations

### JSON-RPC 2.0 Protocol

All agent communication follows JSON-RPC 2.0 over HTTP with proper A2A Message object format:

**Synchronous Call (message/send)**:
```json
{
  "jsonrpc": "2.0",
  "method": "message/send",
  "params": {
    "message": {
      "kind": "message",
      "role": "user",
      "messageId": "uuid",
      "parts": [
        {
          "kind": "text",
          "text": "instruction"
        }
      ]
    }
  },
  "id": "uuid"
}
```

**Async Task Submission (message/stream with SSE)**:
```json
{
  "jsonrpc": "2.0",
  "method": "message/stream",
  "params": {
    "message": {
      "kind": "message",
      "role": "user",
      "messageId": "uuid",
      "parts": [
        {
          "kind": "text",
          "text": "instruction"
        }
      ]
    }
  },
  "id": "uuid"
}
```

**SSE Response** (submit_task extracts taskId from first event):
```
event: status-update
data: {"jsonrpc":"2.0","result":{"taskId":"task-123","state":"submitted"},"id":"uuid"}
```

**Task Status (tasks/get)**:
```json
{
  "jsonrpc": "2.0",
  "method": "tasks/get",
  "params": {"id": "task-123"},
  "id": "uuid"
}
```

### Tool Implementation Pattern

All tools follow the same structure:

1. **Inherit from `dify_plugin.Tool`**
2. **Implement `_invoke(tool_parameters) -> Generator[ToolInvokeMessage, None, None]`** - Main execution logic
3. **Yield `ToolInvokeMessage`** - Use `yield self.create_text_message()` for text responses (not `return`)
4. **Build agents registry at runtime** - Via `_build_agents_registry()` method
5. **Access credentials** - Via `self.runtime.credentials.get("agent_N_field")`
6. **Error handling** - Catch `requests.exceptions.RequestException`, `json.JSONDecodeError`

## File Organization

### Source Files

- **manifest.yaml** - Plugin metadata, version, and entrypoint configuration
- **main.py** - Plugin initialization with DifyPluginEnv (MAX_REQUEST_TIMEOUT=120)
- **provider/a2a.yaml** - Provider definition with credential schema
- **provider/a2a.py** - Provider class with credential validation logic
- **tools/*.yaml** - Tool definitions (identity, parameters, schemas)
- **tools/*.py** - Tool implementation classes

### Documentation Files

- **README.md** - User-facing documentation with installation and usage guide
- **CLAUDE.md** - This file - development documentation for AI assistants
- **PRIVACY.md** - Privacy policy for plugin (referenced in manifest.yaml)
- **LICENSE** - Apache License 2.0

### Asset Files

- **_assets/a2a-logo-black.svg** - Plugin icon (black A2A logo on white circular background)
- **_assets/a2a-logo-white.svg** - Plugin icon (white version)
- **screenshots/** - Documentation screenshots (18 PNG files for README)
  - Installation flow (00-plugins-button through 01-installation-success)
  - Tools browser integration (02-04)
  - Plugin overview (05)
  - Individual tool configurations (06-10)
  - Agent configuration (11-12, 14)

**Note:** manifest.yaml references `icon: a2a-logo-black.svg` but the file is located in `_assets/`. The icon uses a white circular background for visibility on both light and dark interfaces.

### Build Artifacts

- **dify-a2a-plugin.difypkg** - Packaged plugin bundle
  - Created by: `dify plugin package`
  - Should be gitignored (per .gitignore pattern `*.difypkg`)
  - May appear in untracked files during development

- **dify-a2a-plugin/** - Expanded package directory
  - Gitignored (per .gitignore line 3)
  - Created during local development/testing

### Ignored Files (.difyignore)

The `.difyignore` file excludes these from package builds:
- Python bytecode (`__pycache__/`, `*.py[cod]`)
- Virtual environments (`.venv`, `venv/`, `ENV/`)
- IDE files (`.idea/`, `.vscode/`, `.DS_Store`)
- Git files (`.git/`, `.gitignore`, `.github/`)
- Build artifacts (`*.difypkg`, `dify-a2a-plugin/`)
- Test files (`tests/`)
- Documentation assets (`screenshots/`)
- Local reference materials (`dify-plugin-daemon/`, `dify_reference/`)
- Kubeconfig files (`*.kubeconfig`)
- CLAUDE.md (development documentation)

### YAML Schema Files

Tool YAML files (`tools/*.yaml`) define:
- Tool identity (name, label, description, tags)
- Parameters with types, validation, and human/LLM descriptions
- Parameter schemas follow Dify's tool definition format

## Testing

Unit tests are in `tools/tests/test_a2a_registry.py` and use mocking:

```python
# Mock dify_plugin module (not available outside Dify runtime)
mock_dify_plugin = MagicMock()
sys.modules["dify_plugin"] = mock_dify_plugin

# Mock HTTP requests
@patch('requests.post')
def test_call_agent_success(self, mock_post):
    # Test implementation
```

Run tests with: `python3 -m unittest tools/tests/test_a2a_registry.py`

## Plugin Versioning

Version is specified in **two places** (must match):
1. `manifest.yaml` - `version` field (line 2)
2. `manifest.yaml` - `meta.version` field (line 19)

**Current Version:** 0.1.0

When bumping versions, update both fields.

## Common Development Workflows

### Adding a New Tool

1. Create `tools/new_tool.yaml` with tool definition
2. Create `tools/new_tool.py` with Tool class implementation
3. Add tool to `provider/a2a.yaml` under `tools:` section
4. Add unit test in `tools/tests/`
5. Rebuild package with `dify plugin package`

### Modifying Agent Registry Schema

If changing the credential structure in `provider/a2a.yaml`, update:
- `credentials_for_provider` section in `provider/a2a.yaml`
- Validation logic in `provider/a2a.py:_validate_credentials()`
- All tool implementations that parse `agents_registry` via `_build_agents_registry()`

### Local Testing Strategy

Since `dify-plugin` SDK is only available in Dify runtime:
- Use mocking for unit tests (see `tools/tests/test_a2a_registry.py`)
- Test with Dify CLI's debug runtime: `dify plugin start --debug`
- For full integration testing, install in local Dify instance

**Note:** `dify-plugin-daemon` directory (if present locally) is gitignored and not part of the repository.

## Dependencies

Runtime dependencies (from `requirements.txt`):

- **dify-plugin==0.6.2** - Core plugin SDK (runtime dependency)
- **requests>=2.31.0** - HTTP client for A2A protocol calls
- **sseclient-py>=1.8.0** - SSE (Server-Sent Events) client for streaming responses

Development dependencies:
- **Python 3.12** - Required by manifest (runner.version) for production
  - Newer versions (e.g., 3.14) may work in local development
  - Dify runtime will use Python 3.12 as specified in manifest

Build tools:
- **Dify CLI** - Plugin packaging and local testing
  - Install: `brew tap langgenius/dify && brew install dify`

## A2A Protocol Specification

This plugin implements the A2A (Agent-to-Agent) protocol v0.3.0:

- **Agent Discovery**: Via local agent registry configuration
  - Future: May support external A2A registries for dynamic agent discovery
- **Agent Card**: Fetched from `{base_url}/.well-known/agent-card.json` with fallback to `agent.json`
- **Synchronous Messaging**: `message/send` for request/response (waits for full result)
- **Asynchronous Tasks**: `message/stream` (SSE streaming) + `tasks/get` for long-running operations
  - `message/stream` creates a task and streams progress via SSE events
  - `submit_task` tool extracts taskId from first SSE event and returns immediately
  - `tasks/get` retrieves task status and results
- **Authentication**: Supports none, bearer, api-key, and basic auth types
- **Message Format**: Proper A2A Message objects with `kind`, `role`, `messageId`, and `parts[]`

**Note**: The A2A spec does NOT include a `tasks/send` method. Task creation happens via `message/send` or `message/stream`.

## Repository Structure

This plugin is intended for publication in the Dify Marketplace:

- **Source code** - All implementation files
- **Documentation** - README, PRIVACY, CLAUDE.md
- **Assets** - Icons in `_assets/`, screenshots in `screenshots/`
- **Build artifacts** - .difypkg package (gitignored)
- **License** - Apache License 2.0

**Repository URL:** https://github.com/flyryan/dify-a2a-plugin
**Current Version:** 0.1.0
**Developer:** Ryan Duff (ry@nduff.com)

## Additional Notes

### Icon Implementation

The plugin icon (`a2a-logo-black.svg`) uses a black A2A logo on a white circular background. This design ensures visibility on both light and dark interfaces, solving the common problem where pure black or white icons disappear on matching backgrounds.

### Screenshot Organization

The `screenshots/` directory contains 18 PNG files documenting the plugin:
- **Installation flow** (00-01): Complete step-by-step installation process
- **Tools browser** (02-04): Finding and enabling plugin tools
- **Plugin overview** (05): All 5 actions with descriptions
- **Tool configurations** (06-10): Each of the 5 tools with parameters
- **Agent configuration** (11-12, 14): Setting up agents in the registry

These screenshots are referenced in README.md for user documentation but excluded from the plugin package via `.difyignore`.

### Environment Configuration

The `main.py` sets `MAX_REQUEST_TIMEOUT=120` (seconds) for the plugin environment, allowing longer operations for:
- Agent capability discovery (may involve slow endpoints)
- Synchronous agent calls (up to 60 second timeout)
- Task status polling
- SSE stream processing for async tasks

### Future Enhancements

As noted in README.md, the List Agents tool currently lists locally configured agents. Future versions may incorporate support for querying external A2A agent registries as they become standardized, either by:
- Enhancing the existing List Agents tool
- Adding dedicated registry query tools

---
> Source: [flyryan/dify-a2a-plugin](https://github.com/flyryan/dify-a2a-plugin) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-09 -->
