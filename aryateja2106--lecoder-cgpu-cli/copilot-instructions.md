## lecoder-cgpu-cli

> CLI tool for executing code on Google Colab GPUs. **All commands support `--json` for machine-readable output.**

# LeCoder cGPU - AI Agent Reference

CLI tool for executing code on Google Colab GPUs. **All commands support `--json` for machine-readable output.**

## Quick Start

```bash
npm install -g lecoder-cgpu
lecoder-cgpu auth login      # One-time OAuth setup
lecoder-cgpu connect         # Interactive GPU terminal
```

## Commands

### Execute Code

```bash
# Run Python on GPU (kernel mode - recommended)
lecoder-cgpu run -m kernel "import torch; print(torch.cuda.is_available())" --json

# Run shell command
lecoder-cgpu run "nvidia-smi" --json

# Multi-line code (quote properly)
lecoder-cgpu run -m kernel 'for i in range(3):
    print(i)' --json
```

### Check Status

```bash
lecoder-cgpu status --json    # Auth + runtime + GPU info
```

### Notebook Management

```bash
lecoder-cgpu notebook list --json
lecoder-cgpu notebook create "my-notebook" --json
lecoder-cgpu notebook delete <ID> --force --json
```

### Session Management

```bash
lecoder-cgpu sessions list --json
lecoder-cgpu sessions clean              # Remove stale sessions
```

### Execution History

```bash
lecoder-cgpu logs -n 10 --json           # Last 10 executions
lecoder-cgpu logs --status error --json  # Only failures
lecoder-cgpu logs --stats --json         # Summary statistics
```

## JSON Output Structure

### Successful Execution
```json
{"status":"ok","errorCode":0,"stdout":"output\n","timing":{"duration_ms":123}}
```

### Failed Execution
```json
{
  "status": "error",
  "errorCode": 1005,
  "error": {
    "name": "ImportError",
    "message": "No module named 'pandas'",
    "category": "import",
    "suggestion": "pip install pandas"
  }
}
```

## Error Codes

| Code | Category | Action |
|------|----------|--------|
| 0 | SUCCESS | None |
| 1001 | SYNTAX | Fix Python syntax |
| 1002 | RUNTIME | Fix code logic (NameError, TypeError, etc.) |
| 1003 | TIMEOUT | Reduce computation or increase timeout |
| 1004 | MEMORY | Reduce batch size or model size |
| 1005 | IMPORT | Install missing package: `lecoder-cgpu run "pip install <pkg>"` |
| 1006 | IO | Check file paths/permissions |
| 1999 | UNKNOWN | Check logs: `lecoder-cgpu debug tail` |

## Runtime Options

```bash
--tpu          # Request TPU instead of GPU
--cpu          # Request CPU-only runtime  
--new-runtime  # Force new runtime (don't reuse)
```

## Common Patterns

### Install Package Then Run Code
```bash
lecoder-cgpu run "pip install transformers" && \
lecoder-cgpu run -m kernel "from transformers import pipeline; print('OK')" --json
```

### Check GPU Memory Before Large Model
```bash
lecoder-cgpu status --json | jq '.runtimes[0].gpu.memory.free'
```

### Auto-Retry on Transient Errors
```python
import subprocess, json

def run_gpu(code, retries=3):
    for i in range(retries):
        result = subprocess.run(
            ["lecoder-cgpu", "run", "-m", "kernel", "--json", code],
            capture_output=True, text=True
        )
        data = json.loads(result.stdout)
        if data["errorCode"] == 0:
            return data
        if data["error"]["category"] not in ["timeout", "io", "unknown"]:
            break  # Non-retryable
    return data
```

## Debugging

```bash
lecoder-cgpu debug tail              # Recent log entries
lecoder-cgpu debug show -l error     # Errors only
lecoder-cgpu status --json           # Full runtime state
LECODER_CGPU_DEBUG=1 lecoder-cgpu <cmd>  # Verbose output
```

## File Locations

| Path | Contents |
|------|----------|
| `~/.config/lecoder-cgpu/config.json` | OAuth client credentials (client ID/secret) |
| `~/.config/lecoder-cgpu/state/session.json` | User session token (refresh token) |
| `~/.config/lecoder-cgpu/state/history.jsonl` | Execution history |
| `~/.config/lecoder-cgpu/state/logs/` | Debug logs |

> **Note:** On macOS, files are stored in `~/Library/Preferences/lecoder-cgpu/` instead.

## Headless/Container Authentication

OAuth login requires a browser, which doesn't work in Docker containers or CI environments. You need to copy **both** the config file (OAuth app credentials) and the session file (user token).

### Method 1: Copy Full Config Directory (Recommended)
```bash
# From host machine
docker cp ~/.config/lecoder-cgpu container:/root/.config/lecoder-cgpu

# On macOS, use:
docker cp ~/Library/Preferences/lecoder-cgpu container:/root/.config/lecoder-cgpu
```

### Method 2: Export and Import Files
```bash
# On host machine (with browser):
# 1. Get the OAuth config
cat ~/.config/lecoder-cgpu/config.json  # or ~/Library/Preferences/lecoder-cgpu/ on macOS

# 2. Get the session token
lecoder-cgpu auth-export --json
```

### Docker Example (Full Working Setup)
```bash
# Get config and session from host
CONFIG_JSON=$(cat ~/Library/Preferences/lecoder-cgpu/config.json)
SESSION_JSON=$(cat ~/Library/Preferences/lecoder-cgpu/state/session.json)

docker run --rm node:18 bash -c "
  npm install -g lecoder-cgpu &&
  mkdir -p ~/.config/lecoder-cgpu/state &&
  echo '$CONFIG_JSON' > ~/.config/lecoder-cgpu/config.json &&
  echo '$SESSION_JSON' > ~/.config/lecoder-cgpu/state/session.json &&
  chmod 600 ~/.config/lecoder-cgpu/state/session.json &&
  lecoder-cgpu status --json
"
```

### Using auth-import Command
If you already have config.json in the container, you can use auth-import for just the session:
```bash
# In container (after config.json exists):
lecoder-cgpu auth-import '<session JSON>' --force
```

## Tips for Agents

1. **Always use `--json`** for programmatic parsing
2. **Check `errorCode` first** - 0 means success
3. **Use `-m kernel`** for Python code (better error handling)
4. **Parse `error.suggestion`** for actionable fixes
5. **Pre-check GPU** with `status --json` before memory-intensive operations
6. **Retry transient errors** (codes 1003, 1006, 1999) with backoff
7. **Don't retry** syntax (1001) or import (1005) errors without fixing code

## Exit Codes

- `0`: Success
- `1`: Command failed (check JSON for details)
- Non-zero: Various failures

## Help

```bash
lecoder-cgpu --help
lecoder-cgpu <command> --help
```

---
> Source: [aryateja2106/LeCoder-cgpu-CLI](https://github.com/aryateja2106/LeCoder-cgpu-CLI) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-25 -->
