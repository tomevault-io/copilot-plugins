## ps-fuzz

> This project requires Python >= 3.9 (tested with 3.9, 3.10, 3.11).

# Development Setup

## Python Environment

This project requires Python >= 3.9 (tested with 3.9, 3.10, 3.11).

### Setup with uv

1. Create virtual environment with Python 3.11:
   ```bash
   uv venv --python 3.11
   ```

2. Activate the virtual environment:
   ```bash
   source .venv/bin/activate
   ```

3. Install dependencies:
   ```bash
   uv pip install -e ".[dev]"
   ```

### Running Tests

Run all tests:
```bash
pytest
```

Run specific test:
```bash
pytest tests/test_chat_clients.py::TestClientLangChainBaseURL::test_empty_base_url_parameters -v
```

Run tests with verbose output:
```bash
pytest -v
```

---
> Source: [prompt-security/ps-fuzz](https://github.com/prompt-security/ps-fuzz) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-23 -->
