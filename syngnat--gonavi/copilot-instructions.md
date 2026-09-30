## gonavi

> 编码规约正文是仓库根目录的 [`AGENTS.md`](./AGENTS.md)。动手改代码前先读完，并在整个任务中遵守。

# GoNavi — Claude Code

编码规约正文是仓库根目录的 [`AGENTS.md`](./AGENTS.md)。动手改代码前先读完，并在整个任务中遵守。

配套：`.cursor/rules/*.mdc`（Cursor）、`AGENTS.md`（Codex / 通用 Agent）。不要另起一套风格。

语言手册不要混用：

- **Go**：[`GO_STYLE.md`](./GO_STYLE.md)（Uber / Google / Effective Go + 本仓库）。**不是**阿里巴巴 Java 手册。
- **Java**：阿里巴巴 Java 开发手册（仅 `tools/jmx-helper`、`internal/jvm/testdata`）。

硬性底线：

1. 禁止继续膨胀已超标文件（如 `QueryEditor.tsx`、`App.tsx`、`store.ts`、`methods_file.go`、`methods_db.go`）。
2. 能拆包拆包，能拆类拆类，能拆 hook 拆 hook。新逻辑写到新文件，旧上帝文件只接线。
3. 用户可见文案走 `shared/i18n`，禁止手改 `frontend/wailsjs/`。
4. Go 绑定层保持薄，返回 `connection.QueryResult`；驱动实现放 `internal/db`。

---
> Source: [Syngnat/GoNavi](https://github.com/Syngnat/GoNavi) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-30 -->
