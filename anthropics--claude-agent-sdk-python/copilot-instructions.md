## claude-agent-sdk-python

> python -m ruff check src/ tests/ --fix

# Workflow

```bash
# Lint and style
# Check for issues and fix automatically
python -m ruff check src/ tests/ --fix
python -m ruff format src/ tests/

# Typecheck (only done for src/)
python -m mypy src/

# Run all tests
python -m pytest tests/

# Run specific test file
python -m pytest tests/test_client.py
```

# Codebase Structure

- `src/claude_agent_sdk/` - Main package
  - `client.py` - ClaudeSDKClient for interactive sessions
  - `query.py` - One-shot query function
  - `types.py` - Type definitions
  - `_internal/` - Internal implementation details
    - `transport/subprocess_cli.py` - CLI subprocess management
    - `message_parser.py` - Message parsing logic

---
> Source: [anthropics/claude-agent-sdk-python](https://github.com/anthropics/claude-agent-sdk-python) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-04 -->
