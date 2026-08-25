## korean-certified-mail

> Korean certified mail (내용증명) automation skill for Claude Code and Codex.

# korean-certified-mail — Agent Instructions

## Project Overview

Korean certified mail (내용증명) automation skill for Claude Code and Codex.
Supports 14 claim types with court-compliant DOCX output, legal anti-hallucination, and MCP-based PII protection.

## Repository Structure

```
korean-certified-mail/
├── certified-mail/          # Claude Code skill directory (symlinked to ~/.claude/skills/)
│   ├── SKILL.md             # Main skill definition — 9-stage workflow
│   ├── references/
│   │   ├── legal-db.json    # Legal statute/precedent DB by category (source of truth)
│   │   ├── legal-basis.md   # Human-readable allowlist (supplementary)
│   │   └── interview-*.md   # Per-type interview question flows (14 types)
│   └── templates/
│       └── certified-mail.tmpl
├── mcp-server/
│   ├── server.py            # FastMCP server — 10 tools
│   └── requirements.txt     # mcp[cli]>=1.0.0
├── shared/
│   └── certified-mail-docx.py  # DOCX generator (tag-based parser → python-docx)
├── install.sh               # macOS/Linux installer
├── install.ps1              # Windows installer
├── AGENTS.md                # This file
└── README.md
```

## Key Design Decisions

- **Skill directory name**: `certified-mail/` (actual dir) symlinked as `korean-certified-mail` in `~/.claude/skills/`
- **SKILL.md absolute paths**: hardcoded after `install.sh` runs `sed` substitution — do NOT change `{REPO_DIR}` manually
- **MCP server name**: `korean-certified-mail` (matches claude_desktop_config.json key)
- **DOCX tag format**: `[TITLE]`, `[SENDER]`, `[RECIPIENT]`, `[SECTION1]`…`[SECTIONN]`, `[DATE]`, `[SIGNATURE]`, `[ACCOUNT]`
- **PII masking**: real values never reach Claude context — tokens like `SENDER_NAME`, `AMOUNT_3M` used instead

## MCP Tools (server.py)

| # | Tool | Purpose |
|---|------|---------|
| 1 | `list_sessions` | Connection check |
| 2 | `mask_personal_info` | PII masking, session creation |
| 3 | `save_contract` | Token restore + TXT + DOCX save |
| 4 | `load_contract_for_review` | Load existing file with PII masking |
| 5 | `save_reviewed_contract` | Save revised contract |
| 6 | `add_evidence` | Register evidence (text/file/url) |
| 7 | `get_evidence_summary` | List evidence with drafting hints |
| 8 | `verify_legal_citations` | Cross-validate citations vs legal-db.json |
| 9 | `search_law_api` | Law.go.kr Open API (requires LAWGO_API_KEY) |
| 10 | `organize_facts` | Sort facts chronologically + SOL warnings |

## Security Layers

1. **Input sanitize**: 18 injection patterns (Korean + English), control chars, unicode direction chars
2. **PII masking**: names/addresses/amounts → tokens before Claude context
3. **File validation**: path traversal prevention, 200KB limit, line-by-line injection check
4. **Legal anti-hallucination**: only `legal-db.json` allowlist citations permitted, no case numbers

## Development Guidelines

### Adding a new claim type
1. Add interview file: `certified-mail/references/interview-{type}.md`
2. Add category entry to `certified-mail/references/legal-db.json`
3. Add entry to `legal-basis.md` (human-readable)
4. Add row to the interview branch table in `SKILL.md` [3단계]
5. Add type code mapping in `SKILL.md` [9단계] file naming section

### Modifying DOCX output
- Edit `shared/certified-mail-docx.py`
- Font: 바탕 (eastAsia), Times New Roman (latin)
- Page: A4, left=3cm, right=2.5cm, top/bottom=2.5cm
- Line spacing: 200% (`line=480, lineRule=auto`)
- Do NOT change tag format without updating `SKILL.md` [8단계]

### Modifying MCP server
- `server.py` uses FastMCP — add tools with `@mcp.tool()` decorator
- All string inputs MUST pass through `_sanitize_str()` or `_sanitize_dict()`
- All file paths MUST pass through `_validate_output_dir()` or the traversal check block
- Session TTL: 60 minutes (`SESSION_TTL_MINUTES`)

### Legal DB updates
- Edit `certified-mail/references/legal-db.json`
- Only add statutes that are **in force** and **verified**
- Never add case numbers to `statutes[]` — case principles go in `case_principles[]`
- Update `"updated"` date field when modifying

## Environment Variables

| Variable | Required | Description |
|----------|----------|-------------|
| `LAWGO_API_KEY` | No | 법제처 Open API key — enables `search_law_api` tool |

## Testing

```bash
# Syntax check
python3 -m py_compile mcp-server/server.py
python3 -m py_compile shared/certified-mail-docx.py

# DOCX generation test
python3 shared/certified-mail-docx.py /tmp/test.txt

# MCP server start (manual)
python3 mcp-server/server.py
```

## Commit Convention

Labels: `[ADD]` `[FIX]` `[UPDATE]` `[REMOVE]`
Format: `[LABEL] 대상` (1–3 words, Korean OK)
No Co-Authored-By lines.

---
> Source: [kimlawtech/korean-certified-mail](https://github.com/kimlawtech/korean-certified-mail) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-22 -->
