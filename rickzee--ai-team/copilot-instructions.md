## crewai-tools

> CrewAI tool definition and security for ai-team tools


# Tool Definition Pattern

- Use `@tool("Display Name")`; docstring with Args and Returns
- Validate inputs first (path validation, content checks)
- Call `audit_log` for the operation; return only safe result

```python
from crewai.tools import tool

@tool("Read File")
def read_file(file_path: str) -> str:
    """Read a file from the workspace directory securely.

    Args:
        file_path: Relative path to the file within the workspace.

    Returns:
        The file contents as a string.
    """
    validated_path = validate_path(file_path)
    audit_log("read_file", validated_path)
    return validated_path.read_text()
```

For the full security checklist (path traversal, sandboxing, PII/secret detection, etc.), see the security-requirements rule.

---
> Source: [RickZee/ai-team](https://github.com/RickZee/ai-team) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-03 -->
