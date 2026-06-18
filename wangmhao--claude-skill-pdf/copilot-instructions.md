## claude-skill-pdf

> Convert Markdown files to professionally styled PDFs. Use when the user wants to export a .md file as PDF for sharing with clients or external stakeholders. Supports tables, code blocks, CJK (Chinese) characters, multiple templates, and batch conversion.


# Markdown to PDF Converter

Convert Markdown documents to professionally styled PDFs using pandoc + weasyprint with customizable CSS templates.

## Input

The user provides a Markdown file path (or multiple paths / glob): `$ARGUMENTS`

If no path is provided, ask the user which `.md` file to convert.

## Dependencies

- **pandoc** — `brew install pandoc`
- **weasyprint** — `pip3 install --user weasyprint`
- **CSS templates** — bundled in `${CLAUDE_SKILL_DIR}/templates/`

If either tool is not found, print install instructions.

## Arguments

Parse `$ARGUMENTS` for:
- **Input file(s)** — one or more `.md` file paths, or a glob pattern like `*.md` (required)
- **`--output` or `-o`** — custom output path (optional, defaults to `~/Desktop/<filename>.pdf`; single file only)
- **`--title` or `-t`** — document title for HTML metadata (optional, defaults to filename)
- **`--template`** — template name (optional, defaults to `report`)

## Templates

Available templates (CSS files in `${CLAUDE_SKILL_DIR}/templates/`):

| Template | Description |
|----------|-------------|
| `report` | Professional report with dark table headers, zebra striping, page numbers (default) |
| `minimal` | Clean, lightweight — GitHub-style tables, no border accents |
| `branded-example` | Branded example with "visiAI — Confidential" footer |

Users can also pass a path to any `.css` file as the template value.

Template resolution order:
1. If value ends in `.css` and file exists → use directly
2. `comms/templates/<name>.css` (project-local override)
3. `${CLAUDE_SKILL_DIR}/templates/<name>.css` (bundled)

## Conversion Steps

### Step 1: Locate files and parse arguments

Verify input file(s) exist. If a file is not found, use Glob to search for likely matches.

### Step 2: Find CSS template

```bash
SKILL_DIR="${CLAUDE_SKILL_DIR}"

# Resolve template
find_css() {
  local name="$1"
  # Direct .css path
  if [[ "$name" == *.css ]] && [[ -f "$name" ]]; then echo "$name"; return; fi
  # Project-local
  if [[ -f "comms/templates/${name}.css" ]]; then echo "comms/templates/${name}.css"; return; fi
  # Skill-bundled
  if [[ -f "${SKILL_DIR}/templates/${name}.css" ]]; then echo "${SKILL_DIR}/templates/${name}.css"; return; fi
}
CSS=$(find_css "${TEMPLATE:-report}")
```

### Step 3: Convert MD to HTML

```bash
pandoc "<input.md>" \
  -o "/tmp/md2pdf_temp.html" \
  --standalone \
  --metadata "title=<title>" \
  --css "<css-path>" \
  --embed-resources
```

### Step 4: Convert HTML to PDF

```bash
weasyprint \
  "/tmp/md2pdf_temp.html" \
  "<output.pdf>" \
  2>/dev/null
```

### Step 5: Cleanup and report

```bash
rm -f /tmp/md2pdf_temp.html
```

Report:
- Output file path
- File size
- Remind user to open and verify formatting

## Batch Mode

If the user provides multiple files or a glob pattern (e.g., `*.md`):
1. Expand the glob to a list of files
2. Convert each file individually (output to `~/Desktop/<filename>.pdf`)
3. Report results in a summary table:

```
Status  File                                               Size
------  --------------------------------------------------  ----
OK      ~/Desktop/report1.pdf                              245K
OK      ~/Desktop/report2.pdf                              180K
FAIL    broken.md                                          conversion error
```

Alternatively, use the bundled shell script directly:
```bash
bash "${CLAUDE_SKILL_DIR}/scripts/md2pdf.sh" --template report *.md
```

## Shell Script

A standalone `scripts/md2pdf.sh` is included for use outside Claude Code:
```bash
# Single file
bash ~/.claude/skills/pdf/scripts/md2pdf.sh report.md

# With template
bash ~/.claude/skills/pdf/scripts/md2pdf.sh --template minimal report.md

# Batch
bash ~/.claude/skills/pdf/scripts/md2pdf.sh --template report *.md
```

## Troubleshooting

- **weasyprint not found**: Run `pip3 install --user weasyprint` (on macOS with managed Python, add `--break-system-packages`)
- **pandoc not found**: Run `brew install pandoc`
- **Chinese characters garbled**: Ensure CSS has CJK font fallback (`PingFang SC`, `Microsoft YaHei`)
- **Tables overflowing**: CSS should have `table { width: 100%; font-size: 10pt; }`
- **Template not found**: Check `ls ~/.claude/skills/pdf/templates/` for available templates

---
> Source: [wangmhao/claude-skill-pdf](https://github.com/wangmhao/claude-skill-pdf) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
