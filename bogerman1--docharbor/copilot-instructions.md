## docharbor

> Use these rules for GitHub Copilot, coding agents, and PR automation working in this repository.

# DocHarbor GitHub Agent Instructions

Use these rules for GitHub Copilot, coding agents, and PR automation working in this repository.

## Core principles

- Prefer DocHarbor CLI/MCP entrypoints over ad hoc parser scripts.
- Keep document structure internal to DocHarbor. Providers translate text; DocHarbor reconstructs Office, PDF, CAD, manifests, and indexes.
- Preserve compatibility with legacy `proj/<slug>/` flows unless a patch intentionally migrates that surface.
- Add or update tests when changing schemas, provider dispatch, workspace resolution, translation modes, output-style validation, or artifact manifests.
- Do not stage generated corpora, project outputs, translated customer files, `.env`, `.pytest_cache`, or temporary probe scripts.

## Translation defaults

- If the request says "translate to English" and does not explicitly ask for source-plus-target output, use `target_only`.
- If the request says "keep original file type", use delivery output: `--output-mode delivery --output-style target_only`.
- Do not infer `bilingual` or `bilingual_full`. They are explicit user choices.
- `bilingual_full` is not generic bilingual output. Validate it per file format before work starts.
- `translate-path` is the normal entrypoint for path-based user requests. Use `translate` only when project/workspace state already exists.

## Lite mode rules

- `--translation-mode lite` swaps only the rough text backend to DeepL/DeepLX.
- DeepL/DeepLX do not parse or render documents and do not provide structured JSON for glossary/audit/review.
- Audit, review, precise glossary extraction, and prompt-addendum generation must remain on an LLM-capable provider.
- Preflight the backend before large `lite` runs.
- `deeplx` requires a running DeepLX server, normally `http://127.0.0.1:1188/translate`.
- official `deepl` requires `DEEPL_API_KEY` or `DOC_AGENT_TRANSLATE_API_KEY`.
- Treat DeepLX `429`, `503`, and upstream IP-block responses as backend unavailable.
- If you fall back from `lite` to `rough`, report that the final artifacts were not produced by DeepL/DeepLX.
- Do not let `deepl` or `deeplx` inherit OpenAI-compatible base URLs such as MiniMax. Use provider defaults unless an explicit DeepL/DeepLX `--base-url` is supplied.

## Verification checklist

- For DOCX delivery, verify the `.docx` can be opened by `python-docx`.
- For XLSX delivery, verify the `.xlsx` can be opened by `openpyxl`; count untranslated source-language fragments in non-formula cells separately from formula cells.
- For PDF delivery, verify page count and sampled text/rendering.
- For DWG delivery, verify the output remains `.dwg` and the header/signature is intact.
- For folders, produce or preserve a manifest mapping each source file to the delivered artifact and verification status.

## Documentation expectations

- Update `CLI_GUIDE.md` when user-facing command behavior changes.
- Update `AGENTS.md` when agent operating rules change.
- Update `CONTRIBUTING.md` when provider, workspace, test, or contribution contracts change.
- Update `TRANSLATE_FLOW.md` when pipeline wiring changes.

---
> Source: [bogerman1/docharbor](https://github.com/bogerman1/docharbor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-14 -->
