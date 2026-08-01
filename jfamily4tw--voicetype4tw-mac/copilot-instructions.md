## voicetype4tw-mac

> 使用者說「開工」或「我要修改程式」時，先做以下讀取，再開始分析或改檔：

# lean-ctx — Token Optimization

## 開工 Onboarding

使用者說「開工」或「我要修改程式」時，先做以下讀取，再開始分析或改檔：

1. 讀本 repo 的 `CLAUDE.md`、`AGENTS.md`、`AI_MEMORY.md`、`README.md`。
2. 回讀 Cowork 專案記憶：`../../專案/嘴炮輸入法/README.md`、`HANDOFF.md`、`DECISIONS.md`、`PROJECT_GUIDE.md`。
3. 檢查 `openspec/` / `.spectra.yaml` 狀態；非瑣碎變更走 Spectra。
4. 回報：架構理解、目前版本、影響範圍、驗證方式與是否需要 Spectra。

不要要求使用者背完整起手式；只要使用者說 `開工，專案：嘴炮輸入法。我要修改程式。`，就自動執行本流程。

lean-ctx is configured as an MCP server. Use lean-ctx MCP tools instead of built-in tools:

| Built-in | Use instead | Why |
|----------|-------------|-----|
| Read / cat / head | `ctx_read` | Session caching, 6 compression modes, re-reads cost ~13 tokens |
| Bash (shell commands) | `ctx_shell` | Pattern-based compression for git, npm, cargo, docker, tsc |
| Grep / rg | `ctx_search` | Compact context, token-efficient results |
| ls / find | `ctx_tree` | Compact directory maps with file counts |

For shell commands that don't have MCP equivalents, prefix with `lean-ctx -c`:

```bash
lean-ctx -c git status    # compressed output
lean-ctx -c cargo test    # compressed output
lean-ctx -c npm install   # compressed output
```

## ctx_read Modes

- `full` — cached read (use for files you will edit)
- `map` — deps + API signatures (use for context-only files)
- `signatures` — API surface only
- `diff` — changed lines only (after edits)
- `aggressive` — syntax stripped
- `entropy` — Shannon + Jaccard filtering

Write, StrReplace, Delete have no lean-ctx equivalent — use them normally.

## LLM System Prompt Management (v2.9.11+)

### Overview
All LLM classes use a **centralized system prompt system** defined in `llm/prompts.py`. This ensures consistency, simplifies maintenance, and enables language-aware fallback behavior.

### Architecture

**Single Source of Truth**: `llm/prompts.py` contains:
- `SYSTEM_PROMPTS` dict: Language-keyed prompts (zh, en, ja)
- `get_default_system_prompt(language="zh")`: Retrieves prompt for given language, falls back to Chinese if not found

**Fallback Pattern**: All LLM classes use this pattern in `refine()`:
```python
effective_prompt = prompt or get_default_system_prompt(self.language)
```

This ensures:
- Empty prompts from user config automatically use language defaults
- No API calls receive empty/null prompts → prevents hallucinations
- Users don't need to configure prompts per LLM engine

### Supported Languages

| Code | Language | Purpose |
|------|----------|---------|
| `zh` | Chinese | Default prompt for speech refinement (preserves original meaning) |
| `en` | English | Same functionality in English |
| `ja` | Japanese | Same functionality in Japanese |

Default: Chinese (`zh`). Other languages fall back to Chinese.

### Configuration

User config structure:
```yaml
llm_engine: openrouter      # Which LLM to use
llm_prompt: ""              # Optional: if empty, uses default for config.language
language: "zh"              # Language code for prompt selection
```

If `llm_prompt` is empty or missing, the `refine()` method automatically selects the prompt based on `language` setting.

### Adding a New Language

1. Add entry to `SYSTEM_PROMPTS` in `llm/prompts.py`:
   ```python
   SYSTEM_PROMPTS = {
       ...
       "fr": "Affinez la sortie de la reconnaissance vocale...",
   }
   ```
2. Users can now set `language: "fr"` in config
3. No changes needed to individual LLM classes

## OpenSSL Bundling (v2.9.11+)

### Problem Solved
Previous versions required manual OpenSSL installation on target machines. v2.9.11 bundles OpenSSL and automatically rewrites library paths during the build process.

### Build Process

**post_build_fix.py** (called from build_all.sh):
1. Copies arm64 native libssl.3 and libcrypto.3 from system Homebrew into app bundle
2. Uses `install_name_tool` to rewrite all `/opt/homebrew/` references to `@loader_path/`
3. Re-signs modified dylibs with ad-hoc codesigning
4. Verifies no hardcoded paths remain

**Result**: Users can install and run the app without any Homebrew dependencies on target machines.

## MLX Version Pin (v2.9.13+)

### Critical Constraint

The bundled MLX library MUST stay in the version range `>=0.29,<0.30`. **Do NOT run `pip install --upgrade mlx`** without first opening a Spectra change to evaluate the impact.

### Why

- MLX 0.30+ ships PyPI wheels tagged `macosx_26_0_arm64` with a `mlx.metallib` compiled using Metal Shading Language 4.0.
- MSL 4.0 is only loadable on the macOS 26 (Tahoe) Metal driver.
- Build hosts on macOS 26 silently pick up MLX 0.30+ via `pip install`. The resulting `.app` bundle then fails on every macOS 13/14/15 user with `RuntimeError: Unable to load kernel ... using language version 4.0 which is incompatible with this OS`, or a C-level `abort()` from MLX during warmup (cannot be caught by Python try/except, no traceback in debug.log).
- This was discovered after multiple emergency mitigations on 2026-05-13/14 (codesign reseal, entitlements, warmup noop) failed to explain why one user worked but four others crashed. Root cause was always MLX MSL version, not signing or warmup.

### How It Is Enforced

1. `requirements.txt` pins `mlx>=0.29,<0.30` with a comment explaining the rule.
2. `scripts/pre_build_check.py` runs at the start of `build_all.sh`. It reads the installed MLX version and wheel platform tag; the build aborts with a clear error and the exact remediation command if MLX is not in range. Override `_MLX_VERSION_OVERRIDE` env var only for testing the failure path.
3. `post_build_fix.py::check_metallib()` runs after dylib rewrites and before `reseal_bundle()`. It logs the bundled metallib's size and mtime, and warns (does not abort) if `xcrun metal-objdump` reveals MSL 4.0.
4. The full canonical spec is `openspec/specs/mlx-version-pin/spec.md` (or the corresponding archived change `pin-mlx-for-cross-os-compat`).

### How To Recover

If `pre_build_check.py` fails because MLX is too new:

```bash
/Library/Frameworks/Python.framework/Versions/3.12/bin/python3.12 -m pip install 'mlx==0.29.4'
```

Then re-run `bash build_all.sh`.

### When Can We Bump?

Only when macOS 26 has reached majority of the addressable user base AND a Spectra change explicitly justifies dropping macOS 13/14/15 support in its `design.md`.

<!-- KARPATHY-CODING-DISCIPLINE:START -->

## Coding Discipline

- 實作前先鎖定目標、範圍、模糊點與驗證方式。
- 選擇最小足夠修改，優先沿用現有架構與命名。
- 只修改任務範圍內的檔案；不要順手重構、改註解、改格式或移除無關 code。
- 完成前跑最接近變更的測試、lint、typecheck、build 或 smoke check。
- 無法驗證時，在回報中說明 skipped checks、原因與使用者可驗證的下一步。

<!-- KARPATHY-CODING-DISCIPLINE:END -->

---
> Source: [jfamily4tw/voicetype4tw-mac](https://github.com/jfamily4tw/voicetype4tw-mac) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-23 -->
