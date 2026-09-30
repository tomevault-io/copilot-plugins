## drawio-mcp

> Alternative approach that works without installing the MCP server. Users add instructions to a Claude Project that teach Claude to generate draw.io URLs using Python code execution.

# Project Instructions

Alternative approach that works without installing the MCP server. Users add instructions to a Claude Project that teach Claude to generate draw.io URLs using Python code execution.

## Key Files

| File | Purpose |
|------|---------|
| `claude-project-instructions.txt` | Instructions to paste into Claude Project settings |

## How It Works

1. Claude generates diagram code (Mermaid, XML, or CSV)
2. Executes Python code to compress and encode the diagram
3. The script outputs a complete HTML page with the URL embedded as a clickable button
4. Claude presents the HTML as an artifact — the user clicks the button to open draw.io

## XML Reference

The detailed draw.io XML generation reference (edge routing, containers, layers, tags, metadata, dark mode, style properties, XML well-formedness) lives in `shared/xml-reference.md` at the repo root — the single source of truth for all prompts. Users should copy its contents into their Claude Project alongside `claude-project-instructions.txt`.

## Coding Conventions

- **Allman brace style**: Opening braces go on their own line for all control structures, functions, objects, and callbacks.
- Prefer `function()` expressions over arrow functions for callbacks.
- See the root `AGENTS.md` for examples.

## Why HTML Output?

The generated URL contains compressed base64 data. LLMs silently corrupt base64 strings when reproducing them token by token. By having the Python script output a complete HTML page with the link embedded, the URL never passes through Claude's text generation — ensuring the link is always correct.

---
> Source: [jgraph/drawio-mcp](https://github.com/jgraph/drawio-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-30 -->
