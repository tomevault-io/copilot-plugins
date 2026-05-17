## applescript-mcp

> This is the AppleScript MCP (Model Context Protocol) Server - a dual-implementation MCP server that allows AI assistants to execute AppleScript on macOS systems. The server is available in both Node.js and Python implementations.

# CLAUDE.md

## Project Overview

This is the AppleScript MCP (Model Context Protocol) Server - a dual-implementation MCP server that allows AI assistants to execute AppleScript on macOS systems. The server is available in both Node.js and Python implementations.

**Key Capabilities:**
- Execute AppleScript to interact with macOS applications (Notes, Calendar, Contacts, Messages, Mail, etc.)
- Search files using Spotlight/Finder
- Read/write file contents and execute shell commands
- Remote execution via SSH
- Docker support with `host.docker.internal` for Mac host access

## Architecture

### Dual Implementation
The project provides two parallel implementations:

1. **Node.js** (`server.js`): Primary implementation using `@modelcontextprotocol/sdk`
2. **Python** (`src/applescript_mcp/server.py`): Alternative implementation using Python MCP SDK

Both implementations provide identical functionality and should be kept in sync.

### Core Components

**server.js (Node.js)**
- Main entry point: Implements MCP tool `applescript_execute`
- Execution modes:
  - Local: Uses `/usr/bin/osascript` to run AppleScript files
  - Remote: Uses SSH (node-ssh) to execute on remote Mac hosts
- Configuration via CLI args: `--remoteHost`, `--remoteUser`, `--remotePassword`
- Logging via custom Logger class (levels: ERROR, WARN, INFO, DEBUG)

**Key Functions:**
- `executeAppleScript(code, timeout)`: Determines local vs remote execution
- `executeLocalAppleScript(code, timeout)`: Executes on local machine
- `executeRemoteAppleScript(code, timeout)`: Executes via SSH connection

## Development Guidelines

### Code Style
- Keep implementations simple and minimal (currently <100 LOC for core logic)
- Use proper error handling - return error messages rather than throwing
- Always clean up temporary files
- Log important operations at appropriate levels

### When Making Changes

1. **Maintain Dual Implementation Parity**: If you modify the Node.js version, apply equivalent changes to the Python version and vice versa.

2. **Test Both Implementations**: Ensure changes work in both Node.js and Python versions.

3. **Version Synchronization**: Keep versions consistent in `package.json` and `pyproject.toml`.

4. **Backward Compatibility**: This is a published npm package, so maintain backward compatibility when possible.

### Testing Considerations

- **macOS Required**: AppleScript only works on macOS systems
- **Permissions**: May require accessibility permissions for certain operations
- **Remote Testing**: SSH testing requires:
  - macOS with Remote Login enabled
  - Valid SSH credentials
  - Network connectivity

### Common Tasks

**Adding New Features:**
1. Update tool description in both implementations
2. Add new parameters to the schema (use zod in Node.js)
3. Implement the feature in both execution paths (local + remote)
4. Update README.md with examples
5. Test on both local and remote configurations

**Updating Dependencies:**
- Node.js: Update package.json and run `npm install`
- Python: Update pyproject.toml and run `uv lock`

**Publishing:**
- Node.js: `npm publish` (requires npm credentials)
- Python: Follow Python packaging guidelines

## Important Constraints

1. **Security**:
   - AppleScript execution is inherently privileged
   - Remote execution stores credentials in config (consider security implications)
   - No sandboxing of AppleScript code

2. **Platform**:
   - macOS only for local execution
   - Remote execution requires macOS target with SSH

3. **Timeout Handling**:
   - Default: 60 seconds
   - Configurable via tool parameter
   - Timeout does not kill the process, just returns a timeout message

4. **Error Handling**:
   - Errors are returned as text strings (not exceptions)
   - Both stdout and stderr are captured

## File Structure

```
applescript-mcp/
├── server.js              # Node.js implementation (main entry)
├── src/
│   └── applescript_mcp/
│       ├── server.py      # Python implementation
│       └── __init__.py
├── package.json           # Node.js package config
├── pyproject.toml         # Python package config
├── README.md              # User documentation
└── LICENSE                # MIT License
```

## MCP Tool Specification

**Tool Name**: `applescript_execute`

**Parameters:**
- `code_snippet` (required): Multi-line AppleScript code to execute
- `timeout` (optional): Execution timeout in seconds (default: 60)

**Returns**: Text output from AppleScript execution or error message

**Template Variables** (in code_snippet):
- `{{REMOTE_HOST}}`: Replaced with configured remote host
- `{{REMOTE_USER}}`: Replaced with configured remote user
- `{{REMOTE_PASSWORD}}`: Replaced with configured remote password

## Docker Support

The server detects Docker environments and supports `host.docker.internal` as a special hostname to access the Mac host from within a container. This requires:
- SSH enabled on Mac host
- Proper credentials configured
- `--remoteHost host.docker.internal` argument

## Logging

Set `LOG_LEVEL` environment variable to control verbosity:
- `ERROR`: Only errors
- `WARN`: Warnings and errors
- `INFO`: General information (default)
- `DEBUG`: Detailed debugging information

Logs are written to stderr to avoid interfering with MCP protocol communication on stdout.

---
> Source: [peakmojo/applescript-mcp](https://github.com/peakmojo/applescript-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-17 -->
