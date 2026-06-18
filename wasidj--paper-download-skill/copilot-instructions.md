## paper-download-skill

> Download academic papers by DOI or arXiv ID. Use this skill whenever the user wants to fetch, download, retrieve, or get a PDF of a research paper, article, or preprint. Triggers on phrases like "download this paper", "get the PDF for", "fetch the article", "下载论文", "获取全文", or when given a DOI (10.xxxx/...) or arXiv ID (YYMM.NNNNN). Also triggers when the user mentions a paper by title and wants it in their local collection. Falls back from publisher PDF to arXiv preprint automatically.


# Paper Download

Download academic papers by DOI or arXiv ID using two CLI tools: `doi2pdf` and `arxiv-cli`.

## Language Rule

Reply to the user in the SAME language they use.

## When This Skill Triggers

This skill triggers when the user wants to obtain a PDF of an academic paper. Common trigger phrases:

- "download this paper" / "下载这篇论文"
- "get the PDF for DOI 10.xxxx" / "获取全文"
- "fetch the article at arXiv:YYMM.NNNNN"
- "I need the full text of..."
- Any message containing a DOI (`10.\d{4,}/...`) or arXiv ID (`\d{4}\.\d{4,5}`)

This skill handles **downloading only**. After the PDF is saved, if the user wants to ingest it into their knowledge base, hand off to the `ingest` skill (or `pdf-converter` skill for MinerU conversion).

## Prerequisites

Two `uv`-installed CLI tools must be available:

```bash
# Verify installation
doi2pdf --help
arxiv-cli --help
```

If missing, install them:

```bash
uv tool install doi2pdf     # DOI-based download
uv tool install arxiv-cli   # arXiv-based download
```

## Download Workflow

### Step 1: Identify the paper and output directory

Determine from the user's message:
- **DOI** (e.g., `10.1038/s42256-025-01159-8`)
- **arXiv ID** (e.g., `2509.13365`)
- **Title/author** → search arXiv to find the ID
- **Output directory** → default to `raw/papers/` in the project root, or a user-specified path

### Step 2: Generate a filename

Create a descriptive kebab-case filename from the paper's identifying info:

```
{firstauthor}-{year}-{short-title-slug}.pdf
```

Examples:
- `earp-2025-llm-provenance-problem.pdf`
- `yeh-2023-de-novo-luciferase-design.pdf`
- `vaswani-2017-attention-is-all-you-need.pdf`

### Step 3: Try DOI-based download (if DOI available)

```bash
doi2pdf --doi {DOI} -o {output_dir}/{filename}.pdf
```

**Important**: `doi2pdf -o` requires a full file path (not just a directory). If you pass only a directory, it will fail with `IsADirectoryError`.

After download, verify it's actually a PDF:

```bash
file {output_dir}/{filename}.pdf
```

If the result is `HTML document` instead of `PDF document`, the publisher blocked the download (paywall, Cloudflare, etc.). Delete the file and proceed to Step 4.

### Step 4: Fallback to arXiv download

If DOI download fails or no DOI is available, try arXiv:

```bash
# If arXiv ID is known:
arxiv-cli download --id {ARXIV_ID} --dest {output_dir}/ --skip-existing

# If only title/author is known:
arxiv-cli download --title "{title words}" --author "{author}" --dest {output_dir}/ --skip-existing
```

**Note**: `arxiv-cli download` saves files with arXiv's own naming convention (e.g., `2509.13365.pdf`). Rename to match our convention:

```bash
mv {output_dir}/{arxiv_naming}.pdf {output_dir}/{filename}.pdf
```

### Step 5: Verify and report

After successful download:

1. Verify the file exists and is a valid PDF:
   ```bash
   file {output_dir}/{filename}.pdf
   ls -lh {output_dir}/{filename}.pdf
   ```

2. Report to the user:
   - Filename and path
   - File size
   - Page count (if available from `file` output)
   - Source (publisher via DOI, or arXiv preprint)

### Troubleshooting

| Problem | Solution |
|---------|----------|
| `doi2pdf` returns HTML (paywall) | Fallback to arXiv preprint version |
| `doi2pdf` returns 403/Cloudflare error | Try arXiv; try ResearchGate; ask user for institutional access |
| `arxiv-cli` finds no results | Check if the paper is on arXiv at all; try different search terms |
| Paper not on arXiv or open access | Inform user; suggest requesting through interlibrary loan, contacting authors, or checking Sci-Hub |
| File is 0 bytes or corrupted | Delete and retry; check network connection |

## Examples

### Example 1: DOI → Publisher PDF

```
User: "下载 DOI 10.1038/s42256-025-01159-8 到 raw/papers/llm-provenance-transclusion/"

→ doi2pdf --doi 10.1038/s42256-025-01159-8 -o raw/papers/llm-provenance-transclusion/earp-2025-llm-provenance-problem.pdf
→ file check: PDF document ✓
→ Report: Downloaded 245KB PDF to raw/papers/llm-provenance-transclusion/earp-2025-llm-provenance-problem.pdf
```

### Example 2: DOI fails → arXiv fallback

```
User: "下载 DOI 10.1038/s42256-025-01159-8"

→ doi2pdf fails (returns HTML)
→ Search arXiv for "LLM scholarly writing provenance"
→ arxiv-cli download --id 2509.13365 --dest raw/papers/
→ mv raw/papers/2509.13365.pdf raw/papers/earp-2025-llm-provenance-problem.pdf
→ Report: Publisher PDF unavailable, downloaded arXiv preprint v1 (8 pages, 245KB)
```

### Example 3: arXiv ID directly

```
User: "帮我下载 arXiv:2303.08774"

→ arxiv-cli download --id 2303.08774 --dest raw/papers/
→ mv raw/papers/2303.08774.pdf raw/papers/yeh-2023-de-novo-luciferase-design.pdf
→ Report: Downloaded from arXiv, saved as yeh-2023-de-novo-luciferase-design.pdf
```

## Integration with Ingest

After downloading a paper, the user may want to ingest it into their knowledge base. In that case:

1. If the paper is a PDF, use the `pdf-converter` skill first to convert it to Markdown via MinerU
2. Then use the `ingest` skill to process it into the wiki

Offer this next step proactively after a successful download.

---
> Source: [WASIDJ/paper-download-skill](https://github.com/WASIDJ/paper-download-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
