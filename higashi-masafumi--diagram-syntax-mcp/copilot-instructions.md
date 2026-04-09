## diagram-syntax-mcp

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is an MCP (Model Context Protocol) server that provides draw.io and Mermaid diagram syntax information. The server aims to help users understand and create diagrams using both draw.io XML format and Mermaid syntax.

## Development Environment

- **Python Version**: Requires Python 3.13+
- **Package Management**: Uses `uv` (modern Python package manager)
- **Dependencies**: httpx, mcp[cli]

## Common Commands

```bash
# Install dependencies
uv sync

# Run the server
python main.py

# Install in development mode
uv pip install -e .
```

## Project Structure

```
mcp-server/
├── main.py                 # Entry point (currently minimal)
├── pyproject.toml          # Project configuration and dependencies
├── documents/              # Documentation and syntax examples
│   ├── drawio/            # Draw.io examples and templates
│   └── mermaid/           # Mermaid syntax documentation
└── uv.lock                # Dependency lock file
```

## Documentation Resources

### Draw.io Documentation
- Located in `documents/drawio/`
- Contains XML-based diagram examples
- Includes templates for various diagram types:
  - Flowcharts, UML diagrams, Network diagrams
  - Business process diagrams, Cloud architecture
  - Engineering diagrams, Floor plans
- Each file contains XML source code with metadata

### Mermaid Documentation  
- Located in `documents/mermaid/`
- Contains syntax examples for different diagram types:
  - Flowcharts, Sequence diagrams, Class diagrams
  - Gantt charts, State diagrams, Entity relationship diagrams
  - Architecture diagrams, Block diagrams, User journey maps
- Files are in Markdown format with frontmatter metadata

## Key Implementation Notes

- The main.py file is currently a placeholder with basic "Hello World" functionality
- The real value is in the extensive documentation collection under `documents/`
- All documentation files have structured metadata (title, description, category, etc.)
- Draw.io examples are stored as decoded XML with complete mxGraphModel structures
- Mermaid examples include both syntax documentation and working code examples

## MCP Server Context

This server is designed to provide diagram syntax information through the Model Context Protocol, allowing AI assistants to access comprehensive examples and templates for both draw.io and Mermaid diagram creation.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Higashi-Masafumi)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Higashi-Masafumi)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
