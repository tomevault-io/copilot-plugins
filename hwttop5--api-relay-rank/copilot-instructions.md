## api-relay-rank

> - 本仓库是一个 Next.js 15 + React 19 + TypeScript 的 AI中转站监视站点，核心页面是 `/ranking`、`/audit`、`/statement`，站点详情页在 `/stations/[station]`。

# api-relay-rank

## 项目定位
- 本仓库是一个 Next.js 15 + React 19 + TypeScript 的 AI中转站监视站点，核心页面是 `/ranking`、`/audit`、`/statement`，站点详情页在 `/stations/[station]`。
- `scripts/` 下的 Python 脚本负责公开快照抓取、站点数据重建、手动审计与任务编排；`data/site-data.json` 是前端主数据源。
- 当前是单仓库项目，不需要默认寻找子目录 `AGENTS.md`；如果未来出现独立子项目，再就近补充。

## 工作规则
- 先看 `git status`，尊重当前未提交改动；不要回滚、覆盖或重写你没有明确要求修改的文件。
- 和用户交流默认使用中文。
- 变更尽量局部化，避免顺手重构无关模块。
- 生成数据只走脚本，不手工编辑 `data/site-data.json`、`data/_public_fetch/` 或其他派生输出。
- 修改文档时，README 只保留面向人类的快速上手；操作细节优先写在 `docs/data-refresh-notes.md`。
- 如果脚本行为不确定，先看对应 `--help` 或源码，再改文档或代码。

## 常用命令
- `npm install`
- `npm run dev`
- `npm run build`
- `npm run start`
- `npm run site:data`
- `npm run site:announcements`
- `npm run site:tiers`
- `npm run site:audit`
- `npm run site:refresh-manual`
- `npm run site:jobs`
- `python -m unittest tests/test_build_site_data.py`
- `python scripts/run_station_audit.py --help`
- `python scripts/fetch_public_content.py --help`
- `python scripts/validate_refresh_outputs.py --help`

## 代码与验证
- TypeScript、TSX、Python 代码保持仓库现有风格，不要为了对齐通用示例而改成别的格式习惯。
- 修改前端或数据构建逻辑后，优先跑最相关的检查；需要时再补 `npm run build` 和/或 `python -m unittest tests/test_build_site_data.py`。
- 不要把密钥、Cookie、邮箱、本地代理地址或用户路径写进提交内容、文档或输出。
- 如果要更新 `docs/data-refresh-notes.md`，必须同步核对脚本 help 和实际行为，避免保留不存在的参数或状态文件说明。

---
> Source: [hwttop5/api-relay-rank](https://github.com/hwttop5/api-relay-rank) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-11 -->
