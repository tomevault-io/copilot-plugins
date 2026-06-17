## docx-editor

> Full MS Office DOCX capabilities (read, edit, format, tables, rendering). Use for surgical manipulation or batch operations on .docx files.


# DOCX Editor Skill

Entry: `python scripts/docx_agent.py <file> <command> [options]`

## Global Flags
- `--json`: Mandatory for programmatic use. Minified output.
- `--backup`: Automatic .bak creation before changes.
- `--cleanup`: Auto-delete temp files.
- `--verify`: Immediate validation after write.

## Knowledge Base
- [Commands](references/commands.md): Full CLI command list.
- [Workflows](references/workflows.md): Optimization patterns for large docs/tables.

## Quick Reference

| Command | What it does |
|---------|-------------|
| `info` | File size, paragraph/table/word count, metadata |
| `outline` | Heading structure with paragraph indices |
| `full_text` | All paragraphs with style labels |
| `read_section --heading X` | Paragraphs under a specific heading |
| `read_range --start N --end M` | Paragraphs N through M |
| `search --query X` | Find paragraphs matching regex |
| `summary_map` | Compact document overview with estimated pages |
| `replace_text --find X --replace Y` | Formatting-preserving text replacement |
| `insert_paragraph --text X --index N` | Insert after paragraph N |
| `delete_paragraphs --indices 1,2,3` | Remove paragraphs by index |
| `create_table --headers A,B,C` | Create a new table |
| `read_table --table-index 0` | Read table as markdown |
| `modify_cell --table-index 0 --row 1 --col 2 --text X` | Edit a cell |
| `batch --commands file.json` | Atomic multi-op batch (supports `create`, `info`, `set_metadata` etc.) |
| `diff --doc2 other.docx` | LCS paragraph-level diff |
| `undo` | Restore last backup |

## Precision Rules
1. Never read full_text first. Use `info` → `outline` → `summary_map`.
2. Targeted reads: Use `search --compact`, `read_range`, or `read_section`.
3. Verify every edit: Use `--verify` or `read_range` follow-up.
4. Batch operations: Use `batch_execute` for multiple edits to minimize I/O.
5. Always `--backup` before destructive operations.

---
> Source: [omselkara/docx-editor](https://github.com/omselkara/docx-editor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-16 -->
