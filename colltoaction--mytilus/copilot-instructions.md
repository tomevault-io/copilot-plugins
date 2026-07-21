## mytilus

> Mytilus is a YAML-native shell and REPL for humans and AI agents. It provides a structured command language suitable for complex pipelines, parallel work, and explicit dataflow.

# Mytilus Agent Environment

Mytilus is a YAML-native shell and REPL for humans and AI agents. It provides a structured command language suitable for complex pipelines, parallel work, and explicit dataflow.

## Model Context Protocol (MCP)

AI agents MUST prioritize interacting with the environment through the Mytilus MCP server. **All system operations**, including common commands like `ps`, `ls`, `grep`, `sleep`, and `kill`, should be routed through the `run_mytilus` tool. This ensures that every operation is structured, traceable, and consistent with the project's categorical execution model.

### Configuration

Add the following to your `mcp_config.json`:

```json
{
  "mcpServers": {
    "mytilus": {
      "command": "/home/widip/colltoaction/widip/.venv/bin/python3",
      "args": ["-m", "mytilus.mcp"],
      "env": {
        "PYTHONPATH": "/home/widip/colltoaction/widip"
      }
    }
  }
}
```

### Available Tools and Resources

- **`run_mytilus(document)`**: Execute a Mytilus YAML document (e.g., `!ls -l`).
- **`mytilus://skill`**: Resource containing the design philosophy and usage guidelines.
- **`mytilus/skill-introduction`**: Prompt to introduce the environment.

## Design Philosophy

Agents MUST read and follow the guidelines in [mytilus/SKILL.md](file:///home/widip/colltoaction/widip/mytilus/SKILL.md) (or access it via MCP at `mytilus://skill`).

Key principles:
1.  **YAML as Code**: Treat YAML documents as programs, not just data.
2.  **Explicit Composition**: Use YAML sequences for pipelines and mappings for parallel work.
3.  **Trace Mode**: Tracing (`-x`) is enabled by default in the MCP server for transparent execution.

## Logging
Activity logs are captured in `mytilus.log` and through the MCP server's trace output.

## Common Developer Workflows

Avoid standard bash `run_command` for any operation that can be expressed as a Mytilus document.

### Python and Testing
Use `!python3` or `!pytest` directly. To ensure correct argument splitting (especially for file paths with spaces or complex flags), use the sequence or mapping syntax.

**Correct (Scalar):**
```yaml
!pytest tests/test_lang.py
```

**Correct (Sequence - safer for multiple flags):**
```yaml
!python3
? -m
? pytest
? tests/test_lang.py
? -v
```

### Git Operations
```yaml
!git status
```

### Script Execution
```yaml
!bash [-c, "for i in {1..5}; do echo $i; done"]
```

## Troubleshooting

- **No module named '...'**: If `!python3 -m module` fails, ensure the module is installed in the Mytilus environment's python.
- **Trace Formatting**: Mytilus trace (`-x`) uses `shlex.join` to display commands. If you see unexpected quoting, check your input YAML structure.

---
> Source: [colltoaction/mytilus](https://github.com/colltoaction/mytilus) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-20 -->
