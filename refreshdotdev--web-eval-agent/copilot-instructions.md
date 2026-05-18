## logging-standards

> Use the Operative Control Center logging system instead of print statements

# Operative Control Center Logging Standards

When working with the Operative codebase, follow these logging practices:

## Core Logging Principle
- **NEVER** use `print()` statements for logging in production code
- Always use the `send_log()` function from the log server module for all logs

## Importing the Log Function
```python
from webEvalAgent.src.log_server import send_log
```

## Basic Usage Pattern
```python
# ❌ Don't do this
print("Some message")

# ✅ Do this instead
send_log("Some message", "➡️")  # Second parameter is the emoji prefix
```

## Emoji Usage Guidelines
Use appropriate emojis as the second parameter to categorize logs:
- 🚀 - Initialization/startup
- ✅ - Successful operation
- ❌ - Error
- ⚠️ - Warning
- 🔍 - Debug information
- 🖱️ - User interactions
- 🖥️ - Console logs
- 🔗 - URL/navigation related
- ➡️ - Network request
- ⬅️ - Network response
- 📝 - Information/notes
- 🏁 - Completion/final state

If a request is not one of these, find an emoji that fits!

## Benefits of the Logging System
- All logs are visible in the Operative Control Center dashboard
- Logs are organized and categorized with emojis
- Users can easily copy logs for debugging
- Logging doesn't interfere with MCP server stdout/stderr

## Example
```python
# Function that performs some operation
def perform_operation():
    send_log("Starting operation", "🚀")
    try:
        # Do something
        result = process_data()
        send_log(f"Operation completed with result: {result}", "✅")
        return result
    except Exception as e:
        send_log(f"Error during operation: {e}", "❌")
        raise
```

Always ensure logs are meaningful and provide context that helps with debugging or understanding the system state. 

---
> Source: [refreshdotdev/web-eval-agent](https://github.com/refreshdotdev/web-eval-agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-17 -->
