## slm-agents

> This file is the durable repository guidance for Codex. Follow it for all work in this repository unless a newer user instruction in the current conversation overrides it.

# AGENTS.md

This file is the durable repository guidance for Codex. Follow it for all work in this repository unless a newer user instruction in the current conversation overrides it.

## Project

DTU Master Thesis: Agents with Small Language Models. This repository investigates which optimization techniques, including constrained decoding, quantization, LoRA fine-tuning, and RAG, enable SLMs to reliably call tools. The work is framed as expanding the boundary of a cascade LLM architecture.

Primary evaluation targets are BFCL v4 and tau-bench retail. The main implementation is Python, with experiments intended for DTU HPC rather than local GPU execution.

## Codex Operating Rules

- Never add `Co-Authored-By` or any other AI attribution to commits, PRs, generated text, or thesis material. This is an academic thesis and all authorship belongs to the student.
- Never commit `.env`, `.mcp.json`, model weight files, Hugging Face caches, local experiment scratch data, or credentials.
- Project-level Codex settings live in `.codex/config.toml`; Codex skills live in `.codex/skills/`. Keep Codex project configuration and skills versionable when they are intentionally part of the repo.
- Do not commit local Codex session state, tool caches, logs, credentials, or machine-specific files.
- Run experiments on DTU HPC, not locally. The local machine should be used for code edits, lightweight checks, and thesis/document work.
- For GitHub operations such as PRs, issues, and repository files, use the configured Codex GitHub MCP tools when available. In this environment that means the `mcp__github_paulobeckhauser__*` tools.
- Do not use the `gh` CLI or call the GitHub REST API directly unless the MCP server lacks the needed operation.
- If direct GitHub API access is unavoidable, use `GITHUB_TOKEN` from `.env` and never print, log, or commit the token.
- After opening a PR, wait for the automated Gemini review, address the comments, reply to each review comment with the fix commit, and mark threads resolved when the tooling supports it.
- Before editing files, inspect the relevant code and existing patterns. Prefer small, targeted changes over broad rewrites.
- Never revert user changes unless the user explicitly asks for that exact revert.
- Use `apply_patch` for manual file edits when available.
- Use `rg`/`rg --files` for search when available.

## Structure

- `src/` - Constrained decoding pipeline, including vLLM backend, prompts, schema handling, and evaluation entry points.
- `llm_sdk/` - Model wrapper package, including `Small_LLM_Model`.
- `scripts/` - HPC job scripts, evaluation scripts, LoRA training, and experiment utilities.
- `data/` - Input datasets and output results.
- `notebooks/` - Experiment notebooks.
- `docs/` - Research notes, planning, decisions, supervision notes, and result writeups.
- `project_plan/` - Formal project plan and bibliography.
- `thesis/` - LaTeX thesis document.
- `vendor/` - Local editable benchmark dependencies.
- `.codex/` - Codex project configuration and repo-specific skills.

## Development Commands

- Install default dependencies: `uv sync`
- Install HPC dependencies: `uv sync --group hpc`
- Run tests: `make test` or `uv run --group dev pytest tests/ -v`
- Run lint/type checks: `make lint`
- Run strict lint/type checks: `make lint-strict`
- Run the main module: `make run` or `uv run python -m src`
- Clean local caches: `make clean`

## Experiment Commands

Run experiments from the DTU HPC login node on `gbar.dtu.dk` using LSF:

```bash
bsub < scripts/hpc/run_bfcl.sh
bsub < scripts/hpc/run_bfcl_quant.sh
bsub < scripts/hpc/run_bfcl_few_shot.sh
bsub < scripts/hpc/run_bfcl_rag.sh
bsub < scripts/hpc/train_lora_aligned.sh
bsub < scripts/hpc/run_bfcl_ft_aligned.sh
bsub < scripts/hpc/run_bfcl_ft_aligned_no_guided.sh
bsub < scripts/hpc/run_tau_bench.sh
```

Monitor jobs with:

```bash
bstat
bjobs
tail -f logs/<jobid>.out
```

HPC model downloads require `HF_TOKEN` and `HF_HOME` pointing to scratch storage.

## Codex Skills

- Use `.codex/skills/thesis-readable-style/` when improving thesis prose, restructuring thesis sections, or making academic writing clearer.
- Use `.codex/skills/thesis-examiner/` when evaluating a thesis or producing examiner-style feedback.
- Use `.codex/skills/find-skills/` when the task is to discover or install additional agent skills.
- Read a skill's `SKILL.md` before applying it.

## Coding Conventions

- Follow the style already used in nearby files.
- Keep changes focused on the requested task. Avoid broad refactors unless they are necessary for correctness.
- Prefer structured parsing and existing helper APIs over ad hoc string manipulation.
- Add or update tests when changing behavior, shared code, experiment logic, or data transformations.
- Do not modify generated outputs or benchmark result artifacts unless the task explicitly requires it.
- Preserve thesis/research wording unless asked to revise it.
- Keep final responses concise and include verification results or explain why verification was not run.

## Verification

Before finishing a code change, run the narrowest relevant check:

- Python logic changes: `make test`
- Style or typing changes: `make lint`
- Shared or high-risk changes: `make lint-strict` plus `make test`
- Thesis LaTeX changes: build the thesis from `thesis/` using the existing project tooling.
- HPC experiment changes: validate scripts locally only for syntax/path sanity, then run the actual experiments on DTU HPC.

If a verification step cannot be run because it requires HPC, credentials, network access, or unavailable dependencies, state that clearly in the final response.

---
> Source: [pabeckha/slm-agents](https://github.com/pabeckha/slm-agents) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
