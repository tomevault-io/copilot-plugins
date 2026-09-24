## sglang-learning

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

A Chinese-language learning manual for the **SGLang** LLM inference engine, written for engineers preparing for inference-related roles, evaluating engines, or contributing upstream. The repo is **a book made of Markdown**, plus build scripts that emit HTML / PDF / EPUB. The `sglang/` git submodule is a source reference and version baseline for the chapters; it is not built by this repository.

Sister project: `https://github.com/jwzheng96/vllm-learning-book` (same chapter template; chapters are written to be read side-by-side).

## Build & deploy commands

```bash
python3 build_html.py        # → _site/  (HTML site with sidebar, lunr search, Mermaid, KaTeX)
python3 build_pdf_epub.py    # → _site/sglang-learning.{pdf,epub}  (needs pandoc + xelatex)
./deploy_gh_pages.sh <remote-url> [branch]   # manual gh-pages push (CI usually handles this)
```

Override I/O paths with `SGLANG_LEARNING_SRC` / `SGLANG_LEARNING_DST` env vars (both scripts respect them).

First-time clone must pull the submodule: `git submodule update --init --recursive`.

## Hard rules

- **Markdown is the only source of truth.** `_site/` and `sglang-learning-html/` are build outputs and are gitignored — never commit them. CI rebuilds on every push; committed HTML will conflict.
- **Keep source and citations synchronized.** The submodule is pinned to commit `024a7a1031fd7d789076f90e203e3a51b9656cb3` (2026-08-27). When the pin changes, update every affected `file_path:line_number` citation and the README badge/map in the same change.
- **Keep the build scripts in sync.** `build_html.py` and `build_pdf_epub.py` must list the same nine section directories or the PDF will silently skip a part.

## Content architecture

Nine top-level Part directories, each numbered (`NN-name/`) and containing numbered chapter files (`NN-topic.md`). The numeric prefix drives ordering in both the sidebar and the combined PDF. Adding a chapter = drop a new `NN-topic.md` into the right Part directory; both build scripts auto-discover via `sorted(d.glob("*.md"))`.

Part mapping (defined in `build_html.py:SECTIONS_META`):

| Dir | Part | Theme |
| --- | --- | --- |
| `01-overview/` | I | Intro & architecture |
| `02-core-concepts/` | II | Core algorithms (RadixAttention, FSM, scheduling) |
| `03-code-walkthrough/` | III | SGLang source walkthrough |
| `04-optimizations/` | IV | Perf (FlashInfer, spec decoding, quant, CUDA Graph) |
| `05-distributed/` | V | TP / DP / EP / disaggregated |
| `06-frontend-language/` | VI | The SGLang DSL (this is what differs from vllm-learning) |
| `07-hands-on/` | VII | Labs |
| `08-production-deployment/` | VIII | Docker / K8s / SLO / router |
| `09-advanced-features/` | IX | Sampling, structured output, multimodal, LoRA, embedding |

## Chapter template (must follow when authoring)

Every chapter opens with a single blockquote that `build_html.py` detects and styles as `lesson-meta`:

```markdown
# Chapter Title

> **谁该读这一篇？** ...
> **前置阅读：** ...
> **耗时：** N 分钟
> **学完能：** ...
```

Then body (Mermaid, tables, code references). Then three fixed trailing sections, in this order:

```
## 小结
## 自检（3-5 题，自答）
## 下一步（跳转推荐）
```

Reference samples to copy from: `01-overview/01-what-is-sglang.md`, `02-core-concepts/01-radix-attention.md`, `03-code-walkthrough/03-scheduler.md`.

## Citing SGLang source

The book's defining convention: when referring to SGLang code, write `` `path/to/file.py:LINE` `` (e.g. `radix_cache.py:303`). Readers click straight to the line in the pinned submodule. The repo geography table in `README.md` (under "SGLang 仓库地标速查") is the canonical "where does X live in sglang/" lookup; update it whenever you cite a new subsystem.

## HTML build internals worth knowing

`build_html.py` is a single self-contained file (CSS, JS, and template all inlined as Python string literals). When tweaking site appearance or behavior, search the file for the relevant marker — there's no separate templates/ or static/ directory. The renderer uses Python `markdown` with `codehilite`, `fenced_code`, `tables`, `toc`, `attr_list`, `sane_lists`. Custom callouts use `{.callout.tip}` / `{.callout.warn}` / `{.callout.good}` / `{.callout.research}` via `attr_list`.

`build_pdf_epub.py` strips cross-file `[text](xxx.md#anchor)` links (regex `_inline_md_link`) before pandoc — keeps text, drops the broken cross-file ref. Default fonts assume macOS (`PingFang SC`); Linux users must edit `mainfont`/`CJKmainfont` to `Noto Sans CJK SC`.

CI: `.github/workflows/pages.yml` runs `build_html.py` and deploys to Pages on every push to `main`.

---
> Source: [jwzheng96/sglang-learning](https://github.com/jwzheng96/sglang-learning) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-23 -->
