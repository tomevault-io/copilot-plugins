## docs-sync

> Keep docs in sync when adding features or changing behavior


# Documentation sync

When you **add a feature**, **change behavior**, or **rename/remove** user-visible or architectural surfaces, update the matching docs in the **same change** (same PR / session). Do not leave code and docs out of sync.

Canonical matrix: [`docs/ARCHITECTURE.md` § 文檔同步](docs/ARCHITECTURE.md#文檔同步).
Docs ownership / target architecture: [`docs/README.md` § 目標信息架構](docs/README.md#目標信息架構).

## Quick checklist

| Change type | Update at minimum |
|-------------|-------------------|
| New/changed Tauri command or event | `lib.rs`, `src/api/tauri.ts`, `types.ts` if needed, **ARCHITECTURE.md IPC tables** |
| New UI panel, shortcut, setting | **USER_GUIDE.md**, **ARCHITECTURE.md** (modules / 常見修改入口), `SettingsPanel` area in ARCHITECTURE if new setting key |
| New/changed `data/` schema or file | **BUILD_JSON.md** or README data section, **ARCHITECTURE.md** data table, example JSON if applicable |
| Build JSON AI workflow or prompt template | **CREATE_BUILD.md**; **BUILD_JSON.md** if schema changes |
| New Rust module or major frontend folder | **ARCHITECTURE.md** directory / module tables |
| Dev workflow (scripts, npm/cargo commands) | **ARCHITECTURE.md**, **README.md**, **PROJECT_WORKFLOW.md**, **AGENTS.md** if agent-facing |
| Task / sprint / commit workflow | **先 PROJECT_WORKFLOW.md**；再 **AI_AGENT_PROMPTS.md** 開頭、**AGENTS.md**、**CLAUDE.md**、**AI_AGENT_SETUP.md**、**workflow.mdc**；**SPRINTS.md**、**sprints/README.md**（SUMMARY 模板）、task docs |
| Encoding / tooling policy | **ENCODING.md**, `.cursor/rules/encoding.mdc` |
| New doc file under `docs/` | **docs/README.md** index / target architecture; new file starts with `文檔目的` / `不負責` |
| Docs information architecture / SOT ownership | **先 docs/README.md § 目標信息架構**；再 **AGENTS.md**、`.cursor/rules/architecture.mdc`、this file if agent-facing |

## Rules

1. **Same change**: code + docs together; mention doc updates in the summary.
2. **No orphan docs**: if removing a feature, remove or update stale sections (grep doc filenames for old command/event names).
3. **One source of truth**: IPC lists live in **ARCHITECTURE.md**; AGENTS.md / cursor rules only summarize—update ARCHITECTURE first when IPC changes.
4. **User vs dev**: player-facing text → USER_GUIDE; architecture/IPC → ARCHITECTURE; JSON schema → BUILD_JSON.
5. **Workflow state**: idea/task/sprint transitions live in **PROJECT_WORKFLOW.md**; prompt templates live in **AI_AGENT_PROMPTS.md**. Follow the workflow state transition table before editing `BRAINSTORM.md`, `SPRINTS.md`, task docs, or sprint summaries.
6. **Purpose block**: every `docs/**/*.md` should state its `文檔目的` and what it does not own near the top.
7. **No long duplication**: if content belongs to another SOT, link to it instead of copying long rules or tables.
8. **Skip only when**: pure internal refactor with zero behavior/API/UI/data change (state explicitly: "no doc updates needed").

## After editing docs

Run `npm.cmd run check:encoding` (LF, no BOM, final newline).

---
> Source: [yuz9610/POE2Overlay](https://github.com/yuz9610/POE2Overlay) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-04 -->
