## slider-captcha-lab

> Guidance for AI agents working in the `chapter-demo` repo.

# AGENTS.md

Guidance for AI agents working in the `chapter-demo` repo.

## Project layout

- `backend/app/` — 主应用（FastAPI）：`solver/` 识别算法、`track.py` 轨迹生成、
  `model/` 纯 numpy 推理 + 权重、`drivers/` 浏览器驱动抽象、`scenarios/` 各演示目标、
  `mock/` 本地出题与服务端校验复刻、`api/` 路由
- `backend/train/` — 训练侧，只在 `.venv-ml` 里跑，不进 `requirements.txt`
- `backend/tools/` — 基准与验收脚本；`tools/probes/` 是一次性对照实验（见其 README）
- `backend/probe_out/` — 实测分析文档（只发 `.md`，图片与存档不入库）
- `backend/probe_tianai*.py` — 早期一次性探测脚本，用于获取真实 DOM 选择器
- `frontend/` — Vue 3 + Vite 可视化控制台
- `.venv/`、`.venv-ml/`、`.venv314/` — 本地虚拟环境；never commit or modify these
- `.scratch/` — 工作区草稿（字体、缓存、采集题图、第三方源码副本），**不入库**，见 `NOTICE`
- `docs/agents/` — per-repo configuration consumed by the engineering skills

## Agent skills

### Issue tracker

Issues and PRDs live as local markdown files under `.scratch/<feature-slug>/` — this repo has no git remote and no `gh`/`glab` CLI, so external PRs are not a triage surface. See `docs/agents/issue-tracker.md`.

### Triage labels

The five canonical triage roles (`needs-triage`, `needs-info`, `ready-for-agent`, `ready-for-human`, `wontfix`) are used unchanged, recorded as a `Status:` line in each issue file. See `docs/agents/triage-labels.md`.

### Domain docs

Single-context: one `CONTEXT.md` and one `docs/adr/` at the repo root (both created lazily, on demand). See `docs/agents/domain.md`.

---
> Source: [a1952304267/slider-captcha-lab](https://github.com/a1952304267/slider-captcha-lab) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-23 -->
