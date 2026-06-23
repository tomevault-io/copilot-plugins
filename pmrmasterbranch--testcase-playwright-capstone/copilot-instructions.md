## testcase-playwright-capstone

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

### Run the app
```bash
streamlit run app.py
```

### Install Python dependencies
```bash
pip install -r requirements.txt
```

### Install Node/Playwright dependencies
```bash
npm install
npx playwright install chromium
```

### Run a single Playwright spec
```bash
npx playwright test "tests/login.spec.ts" --reporter=list
```

### Run all Playwright specs
```bash
npm test
```

## Architecture

This is a **multi-agent AI system** that reads a PDF requirements document, generates Playwright TypeScript tests, executes them, and loops to fix failures. The Python backend orchestrates everything via LangGraph; Streamlit is the UI.

### Data flow

```
PDF + URL
  ↓
Agent A  →  all_requirements (70+ FRs extracted from PDF in chunks)
              ↓
         User selects feature modules in UI
              ↓
Agent B  →  scans live pages (Playwright subprocess) → generates .spec.ts files in tests/
              ↓
Agent C  →  runs 5 checks per FR → marks failures → triggers retry
              ↓
         if failures && retries_left → back to Agent B (fixing only failed FRs)
         else → END
```

### LangGraph graphs (`graph/workflow.py`)

Two compiled graphs:
- `build_workflow()` — full A→B→C pipeline (not currently used by UI)
- `build_bc_workflow()` — B→C only, used by Phase 2 UI after Agent A already ran

The conditional edge `should_retry_or_end` after Agent C checks `failed_fr_ids` vs `max_retries` to decide `"retry"` (back to Agent B) or `"end"`.

### Shared state (`graph/state.py`)

`AgentState` TypedDict flows through all nodes. Key sections:
- **Config**: `llm_provider`, `api_key`, `base_url`, `pdf_path`, `selected_features`
- **Agent A output**: `all_requirements` (all FRs), `selected_requirements` (filtered to user selection)
- **Agent B output**: `spec_files` (`{module: absolute_path}`), `generated_tests` (`{fr_id: GeneratedTest}`)
- **Agent C output**: `agent_c_results` (`{fr_id: AgentCResult}`), `failed_fr_ids`, `traceability_matrix`
- **Retry control**: `current_retry`, `max_retries`

### Streamlit UI (`app.py`) — two-phase flow

```
phase="config"     → render_phase1()        — upload PDF + URL form
phase="discovery"  → render_discovery()     — Agent A runs, saves requirements/ to disk
phase="selection"  → render_feature_selection() — user picks modules, downloads JSON/CSV
phase="testing"    → render_testing()       — B→C graph streams via .stream()
phase="results"    → render_results()       — traceability matrix + exports
```

`st.session_state.agent_a_state` carries the full Agent A output between phases. The B→C pipeline is assembled in `_run_bc_pipeline()` by filtering `all_requirements` to the user-selected modules.

### Agent A (`agents/agent_a.py`)

Splits PDF text into ~12 000-char chunks, calls LLM on each chunk, deduplicates by `fr_id`. Extracts `FunctionalRequirement` objects with `id`, `feature_name`, `feature_module`, `url_path`, `expected_behavior`, `user_actions`. When called from Phase 1 (discovery), `selected_features=[]` so all FRs are returned in `all_requirements`; `selected_requirements` will be empty until Phase 2 sets it.

### Agent B (`agents/agent_b.py`)

Groups FRs by `feature_module`, scans each page with `tools/page_scanner.py`, then calls the LLM to generate a TypeScript spec file. Saves to `tests/{module}.spec.ts` and stores the absolute path in `state["spec_files"][module]`.

On retry, only processes features that have failing FRs and passes `get_agent_b_retry_prompt` with original code + Agent C fix instructions.

### Locator priority in Agent B prompt

SOURCE 1 (preferred): semantic locators derived from requirement text — `getByLabel`, `getByRole`, `getByText` mapped from phrases like "input field labeled Username".  
SOURCE 2 (fallback): CSS selectors from the live page scan — `#id`, `input[name='x']`.

### Agent C (`agents/agent_c.py`)

Runs 5 checks per feature module:
1. **Missing script** (automated) — checks if FR ID appears in spec code
2. **TypeScript syntax** (Playwright dry-run)
3. **Hallucination** (LLM) — verifies locators exist in the page scan
4. **Missing scenario** (LLM) — checks coverage of `expected_behavior` list
5. **Execution** — runs the spec via `tools/code_executor.execute_spec_file()`

`_determine_overall_result()` priority: missing script → fail, execution fail → fail, ts errors → fail, execution skip → skip, hallucinations/partial coverage → warn, else → pass.

### Code executor (`tools/code_executor.py`)

**Critical**: Playwright treats the spec path argument as a regex. Absolute Windows paths break it. Always convert to a forward-slash relative path before passing to `npx playwright test`:

```python
rel_spec = str(spec_path.relative_to(PROJECT_ROOT)).replace("\\", "/")
command = f'npx playwright test "{rel_spec}" --reporter=json'
```

The JSON reporter is configured in `playwright.config.ts` to write to `reports/playwright_results.json`, but `--reporter=json` on the CLI overrides this and writes to stdout. The executor parses `result.stdout`.

### Page scanner (`tools/page_scanner.py`)

Runs a Playwright script in a **subprocess** (not async) to avoid event-loop conflicts with Streamlit. Extracts inputs, buttons, headings, links, checkboxes, dropdowns from the live DOM and returns CSS locators.

### LLM provider (`tools/llm_provider.py`)

Supports `"gemini"` (Gemini Flash 2.5, free tier with 5s RPM delay between calls) and `"claude"` (Claude Haiku). Both wrapped via LangChain `BaseChatModel`. `invoke_llm_with_retry()` retries up to 3 times with back-off on rate limit errors.

### Requirements persistence

After Agent A runs, extracted requirements are saved to:
- `requirements/extracted_requirements.json`
- `requirements/extracted_requirements.csv`

These are written by `save_requirements_to_disk()` in `app.py` and offered as download buttons in the feature selection phase.

### Playwright config (`playwright.config.ts`)

- `testDir: ./tests` — all generated specs land here
- Single Chromium worker, no Playwright-level retries (retries handled by Agent C loop)
- Reporters: `list` (console), `json` → `reports/playwright_results.json`, `html` → `reports/playwright_html/`

---
> Source: [PmrMasterBranch/TestCase_Playwright_Capstone](https://github.com/PmrMasterBranch/TestCase_Playwright_Capstone) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-23 -->
