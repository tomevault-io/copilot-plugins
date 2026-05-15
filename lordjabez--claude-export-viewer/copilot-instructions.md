## claude-export-viewer

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Is

A Python CLI tool that converts a Claude.ai data export ZIP into a browsable static HTML website. The export ZIP contains `users.json`, `projects.json`, `memories.json`, and `conversations.json`. The output is a flat directory with `index.html`, `style.css`, and `conversations/{uuid}.html`.

## Commands

```bash
# Run against a real export
uv run claude-export-viewer path/to/export.zip -o path/to/output/

# Tests
uv run pytest tests/ -v
uv run pytest tests/test_renderer.py::test_render_message_artifact_create  # single test

# Lint
uv run ruff check src/ tests/
uv run ruff format --check src/ tests/
uv run ruff check --fix src/ tests/ && uv run ruff format src/ tests/  # auto-fix
```

## Architecture

The data flows in one direction: **ZIP → models → renderer → templates → HTML files**.

- **`models.py`** — Pydantic v2 models for all four JSON files. Content blocks use a `ContentBlock` discriminated union on the `type` field (`text`, `thinking`, `tool_use`, `tool_result`, `token_budget`). The `ToolUseContent` model has special handling for artifacts via `is_artifact` / `artifact_input` properties — artifact tool_use blocks have `name="artifacts"` and their `input` dict contains `command`, `type`, `title`, `content`, etc.

- **`loader.py`** — Opens the ZIP, finds JSON files by suffix (handles nested paths), and returns an `ExportData` instance.

- **`renderer.py`** — Converts each `ChatMessage` into an HTML string. The key function is `render_message()` which dispatches to per-block-type renderers. Markdown rendering uses mistune v3 with a custom `_HighlightRenderer` that extends `mistune.HTMLRenderer` (not `BaseRenderer` — this matters because `HTMLRenderer.render_token` unpacks token dicts into method args). Pygments handles code highlighting at build time.

- **`html_builder.py`** — Loads Jinja2 templates, renders index + each conversation page, copies `style.css`. Templates use `{{ message_html | safe }}` since message HTML is pre-rendered by the renderer.

- **`cli.py`** — Thin argparse wrapper.

## Rendering Rules

| Content type | How it renders |
| --- | --- |
| `text` | Full markdown via mistune + Pygments |
| `thinking` | Collapsible `<details>`, last summary as label |
| `tool_use` (artifacts, create/rewrite) | Bordered card with title bar; markdown artifacts rendered, code highlighted |
| `tool_use` (artifacts, update) | Skipped (no full content) |
| `tool_use` (other tools) | Collapsible with message text |
| `tool_result` (artifacts) | Skipped |
| `tool_result` (other) | Collapsible with pre-formatted output, truncated at 2000 chars |
| `token_budget` | Skipped |

## Claude.ai Data Export Format Specification

Complete reference for the ZIP export format produced by Claude.ai's "Export Data" feature.

### ZIP Structure

A standard ZIP containing four JSON files. Files may be at the root or in subdirectories — the loader finds them by suffix match (e.g. `data/users.json` works). Missing files are treated as empty arrays.

```text
export.zip
├── users.json          # Array of User
├── projects.json       # Array of Project
├── memories.json       # Array of Memories
└── conversations.json  # Array of Conversation
```

### users.json

```json
[
  {
    "uuid": "string (required)",
    "full_name": "string (required)",
    "email_address": "string (required)",
    "verified_phone_number": "string | null (optional)"
  }
]
```

### projects.json

```json
[
  {
    "uuid": "string (required)",
    "name": "string (required)",
    "description": "string (optional, default '')",
    "is_private": "bool (optional, default false)",
    "is_starter_project": "bool (optional, default false)",
    "prompt_template": "string (optional, default '')",
    "created_at": "ISO 8601 datetime | null (optional)",
    "updated_at": "ISO 8601 datetime | null (optional)",
    "creator": "object | null (optional, typically {uuid: string})",
    "docs": [
      {
        "uuid": "string (required)",
        "filename": "string (required)",
        "content": "string (optional, default '')"
      }
    ]
  }
]
```

### memories.json

```json
[
  {
    "project_memories": {"memory_id": "memory text (string→string map, default {})"},
    "account_uuid": "string (optional, default '')"
  }
]
```

### conversations.json

```json
[
  {
    "uuid": "string (required)",
    "name": "string (optional, default '')",
    "summary": "string (optional, default '')",
    "created_at": "ISO 8601 datetime | null (optional)",
    "updated_at": "ISO 8601 datetime | null (optional)",
    "account": {"uuid": "string"} | null,
    "project_uuid": "string | null (optional)",
    "chat_messages": ["ChatMessage (see below, default [])"]
  }
]
```

### ChatMessage

```json
{
  "uuid": "string (required)",
  "text": "string (optional, default '')",
  "content": ["ContentBlock (see below, default [])"],
  "sender": "'human' | 'assistant' (required)",
  "created_at": "ISO 8601 datetime | null (optional)",
  "updated_at": "ISO 8601 datetime | null (optional)",
  "attachments": [
    {
      "file_name": "string (default '')",
      "file_size": "int bytes (default 0)",
      "file_type": "string (default '')",
      "extracted_content": "string (default '')"
    }
  ],
  "files": [
    {
      "file_name": "string (default '')"
    }
  ]
}
```

Note: `text` is a flattened plaintext version of the message. `content` is the structured block array used for rendering.

### Content Blocks

Discriminated union on the `type` field. A message's `content` array can contain any mix of these.

#### type: "text"

```json
{
  "type": "text",
  "text": "string (default '')",
  "citations": ["object (default [])"]
}
```

#### type: "thinking"

```json
{
  "type": "thinking",
  "thinking": "string (default '')",
  "summaries": [
    {"summary": "string (default '')"}
  ],
  "cut_off": "bool (default false)",
  "alternative_display_type": "string | null (optional)"
}
```

**Quirk**: `summaries` is an array of `{summary: string}` objects, not plain strings.

#### type: "tool_use"

```json
{
  "type": "tool_use",
  "id": "string | null (optional)",
  "name": "string (default '')",
  "input": "object (default {})",
  "message": "string | null (optional)",
  "integration_name": "string | null (optional)",
  "display_content": "string | object | null (optional)"
}
```

When `name` is `"artifacts"`, the `input` object is an artifact descriptor:

```json
{
  "command": "'create' | 'update' | 'rewrite' (default 'create')",
  "id": "string (default '')",
  "type": "string | null (artifact MIME type)",
  "title": "string | null",
  "language": "string | null (programming language)",
  "content": "string | null (full content for create/rewrite)",
  "old_str": "string | null (for update command)",
  "new_str": "string | null (for update command)",
  "version_uuid": "string | null"
}
```

**Artifact MIME types observed**: `text/markdown`, `text/html`, `application/vnd.ant.code`, `application/vnd.ant.react`, `application/vnd.ant.mermaid`, `image/svg+xml`

**Artifact commands**:

- `create` — New artifact. Has `content`, `type`, `title`, `language`.
- `rewrite` — Full replacement. Same fields as `create`.
- `update` — Partial edit. Has `old_str` and `new_str` instead of `content`.

#### type: "tool_result"

```json
{
  "type": "tool_result",
  "tool_use_id": "string | null (optional, links to tool_use.id)",
  "name": "string (default '')",
  "content": "string | [{type: 'text', text: string}] | null (any)",
  "is_error": "bool (default false)",
  "structured_content": "object | null (optional)",
  "message": "string | null (optional)",
  "integration_name": "string | null (optional)",
  "display_content": "string | object | null (optional)"
}
```

**Quirk**: `content` can be a plain string, a list of `{type, text}` objects, `null`, or other structures. The renderer handles string and list-of-objects forms.

#### type: "token_budget"

```json
{
  "type": "token_budget"
}
```

Marker block with no data. Skipped during rendering.

### Known Quirks and Edge Cases

These were discovered by validating against real export data:

- `ThinkingContent.summaries` contains `[{summary: str}]` objects, not plain strings
- `ToolUseContent.id` and `ToolResultContent.tool_use_id` can be `None`
- `display_content` on both tool_use and tool_result can be either a string or a dict (rich_link objects like `{type: "rich_link", link: {url: "..."}}`)
- Artifact `input` has `command` field: `create` has full content, `update` has `old_str`/`new_str`, `rewrite` has full content
- Artifact types seen: `text/markdown`, `text/html`, `application/vnd.ant.code`, `application/vnd.ant.react`, `application/vnd.ant.mermaid`, `image/svg+xml`
- `ToolResultContent.content` can be a string, a list of `{type, text}` objects, or `null`
- `project_uuid` and `account` on conversations are both optional/nullable
- `ChatMessage.text` and `ChatMessage.content` coexist — `text` is a flattened plaintext form, `content` is the structured block array
- ZIP files may have JSON files in subdirectories; the loader matches by filename suffix
- Missing JSON files in the ZIP are treated as empty arrays (partial exports are valid)

---
> Source: [lordjabez/claude-export-viewer](https://github.com/lordjabez/claude-export-viewer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-09 -->
