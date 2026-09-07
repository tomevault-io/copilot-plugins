## seraphine-music

> 以 Markdown 文件形式存放在 `.scratch/` 目录中，每个功能一个子目录。详见 [`docs/agents/issue-tracker.md`](docs/agents/issue-tracker.md)。

## Agent skills

- 使用英文思考,使用中文回答

### Issue tracker

以 Markdown 文件形式存放在 `.scratch/` 目录中，每个功能一个子目录。详见 [`docs/agents/issue-tracker.md`](docs/agents/issue-tracker.md)。

### Triage labels

使用默认五标签映射（needs-triage / needs-info / ready-for-agent / ready-for-human / wontfix）。详见 [`docs/agents/triage-labels.md`](docs/agents/triage-labels.md)。

### Domain docs

单上下文布局，根目录 `CONTEXT.md` + `docs/adr/`。详见 [`docs/agents/domain.md`](docs/agents/domain.md)。

## 项目架构

- 三窗口隔离架构（主窗口 / 桌面歌词 / 迷你播放器），共享基础层；
- 前端 Vue 3 + Pinia，后端 Tauri v2 + Rust。
- 详见 [docs/architecture.md](docs/architecture.md)。

## 关键约定

- 子窗口禁止 import 主窗口 store，数据同步走 Tauri events
- `setup` 中异步任务用 `tauri::async_runtime::spawn`，禁止 `tokio::spawn`
- 事件命名添加业务前缀（如 `music:`）
- Commit 遵循 Conventional Commits
- 详见 [docs/conventions.md](docs/conventions.md)

## 测试验收

- 前端测试：`pnpm test`（Vitest + happy-dom，测试文件在 `tests/` 镜像目录结构，覆盖 utils/stores/components/composables/子窗口/集成测试）
- Rust 测试：`cd src-tauri && cargo test`（纯函数 / serde / 边界 / 常量 / command 符号）
- 类型检查：`pnpm build`（vue-tsc --noEmit，strict 模式）
- 详见 [docs/testing.md](docs/testing.md)

---
> Source: [burenLee/seraphine-music](https://github.com/burenLee/seraphine-music) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-06 -->
