## fastled-wasm

> **ALWAYS use `uv` to run Python code, NOT `python` or `python3`!**

# Cursor Rules for FastLED WASM Project

## CRITICAL: Python Command Usage

**ALWAYS use `uv` to run Python code, NOT `python` or `python3`!**

### Command Guidelines:
- ✅ Use: `uv run <script.py>`
- ✅ Use: `uv run -m <module>`
- ✅ Use: `uv run python <script.py>` (if explicitly needed)
- ❌ NEVER use: `python <script.py>`
- ❌ NEVER use: `python3 <script.py>`

### Examples:
- Instead of `python src/fastled/cli.py`, use `uv run src/fastled/cli.py`
- Instead of `python -m pytest`, use `uv run -m pytest`
- Instead of `python3 build_exe.py`, use `uv run build_exe.py`

### Rationale:
- This project uses `uv` for Python package and environment management
- Using `python` or `python3` directly may use the wrong Python version or miss dependencies
- `uv` ensures consistent execution with the project's specified Python version and dependencies

### Testing:
- Follow the user rule: run unit tests with `bash test`
- For manual testing, use `uv run` prefix for any Python commands

### Development Workflow:
1. Use `uv` for all Python execution
2. Ensure virtual environment is managed by `uv`
3. Install dependencies with `uv add <package>`
4. Run scripts with `uv run <script>`

## Emoji/Emoticon Usage

**ALWAYS use the `EMO()` function for emojis, NEVER hardcode emojis directly!**

### Guidelines:
- ✅ Use: `from fastled.emoji_util import EMO` then `EMO('⚠️', 'WARNING:')`
- ✅ Use: `from fastled.emoji_util import safe_print` for Unicode-safe printing
- ❌ NEVER use: hardcoded emojis like `print("⚠️ Warning")`
- ❌ NEVER use: raw Unicode without fallback handling

### Common Patterns:
- Warning messages: `EMO('⚠️', 'WARNING:')`
- Success messages: `EMO('✅', 'SUCCESS:')`
- Error messages: `EMO('❌', 'ERROR:')`
- Info messages: `EMO('ℹ️', 'INFO:')`
- Playwright/browser: `EMO('🎭', '*')`
- Package/install: `EMO('📦', '*')`

### Rationale:
- Windows cmd.exe has poor Unicode support and will crash on raw emojis
- The `EMO()` function provides fallback text for systems that don't support emojis
- Ensures consistent behavior across all platforms (Windows, macOS, Linux)
- Prevents application crashes due to encoding issues

### Examples:
```python
# Good - with EMO function
from fastled.emoji_util import EMO
print(f"{EMO('⚠️', 'WARNING:')} Debug mode detected with Playwright installed")

# Bad - hardcoded emoji
print("⚠️ Debug mode detected with Playwright installed")
```

## Additional Rules:
- When suggesting Python commands, always prefix with `uv run`
- When creating scripts that execute Python, use `uv run` in the script
- When documenting Python usage, emphasize `uv run` syntax
- Update any existing scripts that use `python` or `python3` to use `uv run`
- When adding user-facing messages with emojis, always use the `EMO()` function

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/zackees) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
