## resume-jd-tailor

> Tailor a resume to a job description (JD) by selecting best-matching work, internship, and research experiences, optionally rewriting bullets with concurrent LLM calls using JD context from the project-root .env, fitting bullet counts to single-line layout budgets, generating JD-specific resume JSON, and rendering outputs through a single HTML-first pipeline that exports directly to PDF with rollback on failure.


# Resume JD Tailor

## Quick start

This skill now keeps only one supported output path: `JSON -> HTML -> PDF`.

1) Run full pipeline and publish latest outputs (recommended):

```powershell
python .\scripts\run_latest.py `
  --lang zh `
  --experiences .\assets\data\experiences\experiences_zh.example.yaml `
  --base .\assets\data\profile\base_resume.example.json `
  --jd .\assets\data\jd\jd.example.yaml `
  --out-dir .\outputs\final `
  --prefix resume `
  --no-llm
```

Default behavior:

- LLM rewriting is enabled by default. Disable with `--no-llm`.
- If `--jd-raw` points to an image (`.png/.jpg/.jpeg/.webp`), the rewrite pass uses Gemini on Vertex AI.
- If `--jd-raw` points to a text file, the rewrite pass uses DeepSeek chat completions.
- The script publishes stable filenames (`resume.pdf/html/json`) and archives all other `resume*` files under `archive/`, so the output directory always shows the latest run.

Public repo data rules:

- This repo tracks only `*.example.*` under `assets/data/`.
- Put your real resume profile/experiences/JD config into `*.local.*` files and keep them untracked via `.gitignore`.
- Never commit `.env` or `outputs/`.

2) Build JD-specific resume JSON (advanced / manual):

```powershell
python .\scripts\build_resume_data.py `
  --lang zh `
  --experiences .\assets\data\experiences\experiences_zh.example.yaml `
  --base .\assets\data\profile\base_resume.example.json `
  --jd .\assets\data\jd\jd.example.yaml `
  --out .\outputs\final\resume.zh.jd.json `
  --no-llm
```

If `.env` contains `DEEPSEEK_API_KEY`, the JSON build step can rewrite selected bullets with LLM calls in parallel so the wording is tightened toward the target JD while staying grounded in the stored experience library.

2) Render HTML and export PDF with automatic fallback:

```powershell
python .\scripts\build_resume_html_pdf.py `
  --data .\outputs\final\resume.zh.jd.json `
  --html-out .\outputs\final\resume.html `
  --pdf-out .\outputs\final\resume.pdf
```

3) Optional visual QA audit of the final PDF with Gemini on Vertex AI (run through `uv`):

```powershell
uv sync --extra gemini

uv run .\scripts\audit_resume_pdf_style.py `
  --pdf .\outputs\final\resume.pdf `
  --vertex-project-id YOUR_GCP_PROJECT_ID `
  --vertex-location global `
  --model gemini-3-flash-preview `
  --out .\outputs\final\resume.style_audit.gemini.json
```

## Fallback behavior

- If the exported PDF exceeds the page limit, the command fails and restores the previous successful HTML/PDF outputs.

## Operating rules

- Prefer editing content in `assets/data/` and regenerating JSON; avoid hand-editing generated `.json`.
- Keep bullet selection deterministic: adjust `assets/data/jd/*.yaml` instead of manually reordering generated bullets.
- Optional LLM rewriting is allowed only as a wording pass. It must keep all claims faithful to the source experience data and should only sharpen emphasis toward the current JD.
- Keep API credentials in `.env` (for example `DEEPSEEK_API_KEY`, `DEEPSEEK_API_BASE`, `DEEPSEEK_MODEL`). `scripts/build_resume_data.py` first checks a repo-adjacent `.env`, then falls back to upward search if needed.
- The JSON build step may issue multiple LLM requests concurrently across selected bullets to reduce latency. Preserve that behavior when extending the script.
- The current template separates `工作经历`, `实习经历`, and `科研经历`. Keep two internship entries max, with one single-line description each.
- Control layout in the data-build step first: trim bullet counts by section-level budgets so the final PDF stays at exactly 1 page.
- The template allows automatic wrapping; do not force hard truncation just to keep each bullet on one line.
- For layout QA, use `scripts/audit_resume_pdf_style.py` with `uv run`. The Gemini review is for visual delivery quality only (blank space, overlap, alignment, clipping, print readiness), not factual resume validation.
- Keep the final PDF at exactly 1 page. If it overflows, shorten bullet text or reduce JD bullet counts and rerun.
- Treat `assets/templates/template_html.html` as the single supported template entry for resume output.

## References

- Template and data conventions: `references/conventions.md`
- Dependencies / troubleshooting: `references/requirements.md`

---
> Source: [Jerry-poor/resume-jd-tailor](https://github.com/Jerry-poor/resume-jd-tailor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
