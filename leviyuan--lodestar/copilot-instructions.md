## lodestar

> <!-- Generated: 2026-05-15 | Updated: 2026-06-13 -->

<!-- Generated: 2026-05-15 | Updated: 2026-06-13 -->

# Lodestar 2.0

## Purpose
本仓库实现一个 Bun daemon，把飞书群消息桥接到无头 `codex app-server` 进程。运行时关系是一个飞书群对应一个 Lodestar session、一个 Codex thread，以及每轮对话中的一张流式 Feishu Card Kit 卡片；项目主群还可用 `model` 管理模型/effort，用 `wt` 自动创建/加入同级 Git worktree 群，用 `agy <prompt>` 启动一次性外部 agy 任务，并用 `task` 启用飞书任务清单自动化。

## Key Files
| File | Description |
|------|-------------|
| `daemon.ts` | daemon 主入口；负责 PID guard、Lark `WSClient`、事件分发、裸词控制命令和 debug socket。 |
| `cli.ts` | npm 分发入口；在缺少 `config.toml` 时触发安装向导，否则延迟导入 `daemon.ts`。 |
| `package.json` | Bun/Node 打包脚本、发布元数据、二进制入口和依赖声明。 |
| `README.md` | 用户安装、首次配置、群控裸词、`model` 选择、`wt` worktree 群、`task` 任务清单自动化和 HTTP 通知端点说明。 |
| `CHANGELOG.md` | 中文发布记录；release notes 需保持同一风格，优先描述用户可感知变化。 |
| `bun.lock` | Bun 依赖锁文件；更新依赖后同步提交。 |
| `LICENSE` | MIT 许可证。 |
| `promo.jpg` | README 顶部展示图。 |

## Subdirectories
| Directory | Purpose |
|-----------|---------|
| `src/` | daemon 的核心 TypeScript 模块，包括 session、Codex 子进程、飞书 API、Card Kit、任务清单 worker 和 CLI 辅助逻辑（见 `src/AGENTS.md`）。 |
| `scripts/` | 面向真实飞书环境的 smoke、调试注入、Card Kit 探针和安装后向导脚本（见 `scripts/AGENTS.md`）。 |

## For AI Agents

### Working In This Directory
- Runtime 是 **Bun**；源码开发通常用 `bun daemon.ts` 或 `bun run start`，发布包通过 `bun build --target=node` 生成 Node 可执行文件。
- daemon 只驱动 `codex app-server --listen stdio://` 的 app-server JSON-RPC 协议；不要恢复 tmux、JSONL 队列或 1.x 传输机制。
- 运行状态全部在 XDG 目录外置：配置默认在 `~/.config/lodestar/config.toml`，日志和 runtime map 默认在 `~/.local/share/lodestar/`。凭据只应存在于 `config.toml`，不要写入仓库。
- Assistant 正文和 footer 状态不使用 Card Kit `/content` 打字流；正文按完整段 `addElement` 插入，footer 状态用 `replaceElement` 直接替换。
- API 失败要记录并向用户暴露；不要静默切换传输、卡片或消息通道作为“兜底”。
- 不要主动重启正在运行的 daemon，除非用户在**当前用户消息**里明确要求 `restart` / `重启` / reload。代码变更后只报告需要重启。
- 停止、重启、替换、shadow、切换或并行接管正在运行的 daemon / user service 的授权**只在当前 assistant 回合内一次性有效**，不得跨用户消息、跨中断恢复、跨上下文压缩或跨任务范围沿用；一旦用户发来新的消息，即使上一条消息要求过“重启”，后续也必须重新明确授权后才能再次操作 live service。
- **禁止**为了“测试”“预览”“发一张看看”“先验证一下”这类目的而停止、重启、替换、shadow、切换或并行接管正在运行的 daemon / user service。只有用户在当前用户消息中**明确点名**要执行对应操作（例如 `systemctl --user restart feishu-daemon.service`、停止当前 daemon、切换到某个 worktree daemon）时才可动手；任何泛化的“测一下”“发测试卡”都**不构成授权**。
- 群内裸词控制是 `hi`、`stop`/`st`、`kill`/`kl`、`restart`/`rs`、`clear`/`cl`、`compact`/`cm`、`model`/`md`、`task`、`wt`/`worktree` 和 `wt <name>`/`worktree <name>`；`agy <prompt>` 启动外部一次性 agy 任务。这些词在 `Session.runCommand` 中作为保留字处理。
- `model` 通过 Card Kit 按钮先选 Codex 模型、再选 reasoning effort，并把选择按 session 持久化到 XDG data。
- `wt <name>` 约定创建同级目录 `<project>[<name>]` 和本地分支 `work/<name>`，并自动创建/加入同名飞书群；解散按钮会先拒绝仍在运行的对应 session，只在 worktree 干净时删除目录和解散群，保留分支；重新激活已合并归档分支时会更新到主线。
- `agy <prompt>` 在当前 session 工作目录内独占运行 `agy --print`，用独立 Card Kit 卡片展示 prompt、状态、输出、仓库变更和“转 Codex”按钮；不要把它混入普通 Codex turn 卡片。
- `task` 打开项目任务清单面板；启用后创建/绑定 `<project>[lodestar]` 飞书任务清单，daemon 内置 worker 会扫描 `设计中`、`[AI]待执行`、`[AI]执行中`、`[AI]待审核`、`已完成` 分组并驱动规划、执行、审核和本地合并。
- 本地脚本可通过 `POST http://127.0.0.1:9876/notify` 发送 `{project, text, level}` 到绑定群。

### Testing Requirements
- 常规校验使用 `bun test` 和 `bun run build`。
- 涉及真实飞书、Codex 登录、卡片流式行为或 `wt` 建群/解散时，用 debug 注入、`bun scripts/smoke.ts "<group name>"` 或 `bun scripts/test-all.ts "<group name>"` 做人工 smoke。
- 需要验证 live 群里的卡片外观或交互时，优先使用**不影响正在运行 daemon** 的路径；如果做不到，先向用户说明会影响哪些服务，并等待明确许可。不得把“为了验证”当成默认可以碰 live daemon 的理由。
- 发布前按项目惯例运行 `bun test`、`bun run build`，再执行版本 bump、tag、npm/GitHub Packages 发布和 GitHub Release 流程。

### Common Patterns
- 根入口保持很薄：`cli.ts` 处理首次配置和 PID guard，`daemon.ts` 负责 WS/event loop，核心业务下沉到 `src/`。
- `Session` 是一个群的状态机；跨群状态只通过 session registry、持久 map、Feishu chat 绑定、模型选择 map 和 `work/*` 分支约定协调。
- `cardkit` 负责每张卡的 sequence、队列、限流和写失败检测；session 负责什么时候开卡、换卡、关闭卡。
- `agy` 任务由 session 管理生命周期，`src/agy-task.ts` 负责 CLI/Git 快照，`src/cards/agy.ts` 负责卡片结构和输出清理，Card action 再把结果转发给 Codex。
- `task` 自动化由 `src/tasklist.ts` 持久化绑定和状态，`src/tasklist-worker.ts` 调度 Codex/agy 子进程，`src/cards/task.ts` 只渲染启用/删除面板。
- 所有 shell 命令卡片展示依赖第一行 `# desc:` 风格说明，修改相关展示逻辑时同步看 `src/cards/turn.ts`。

## Dependencies

### Internal
- `daemon.ts` 依赖 `src/session.ts`、`src/feishu.ts`、`src/config.ts`、`src/paths.ts`、`src/notify.ts` 和 `src/tasklist-worker.ts` 完成启动、事件路由、本机通知和任务清单轮询。
- `src/cards.ts` 是卡片模板 barrel，`src/session.ts`、`src/session-*` 辅助模块和 agy 卡片流程都通过它访问 Card Kit schema。
- `scripts/` 直接导入 `src/` 模块执行真实环境测试，运行前需要有效 `config.toml`。

### External
- `@larksuiteoapi/node-sdk`：飞书/Lark `Client`、`WSClient` 和事件分发。
- Bun：源码运行、测试和构建。
- Node.js >= 18：发布包运行环境。
- `codex` CLI：需要已通过 ChatGPT 登录，daemon 会启动 `codex app-server`。
- Feishu Open Platform：IM、群创建/解散、群成员读写、reaction、附件、Card Kit v1、Task v2、tenant token API。
- systemd user service：长期运行部署时常用，但只有用户明确要求时才操作。

<!-- MANUAL: Add manually maintained notes below this line. -->

## UI Design Notes
- Card Kit 里的操作按钮要优先按手机窄屏设计；高频、重复出现的选择类按钮文案必须尽量短，`model`/effort 这类选择按钮固定用单字 `选`，不要写成 `选择`、`重选` 等多字按钮。
- 生产路径使用 `WSClient + EventDispatcher` 接收 `card.action.trigger`；需要 3 秒内立即更新 JSON 卡片时必须 return `{ card: { type: "raw", data: newCard } }`，不要 return 裸卡片 JSON 或 `{ card: newCard }`；不要在回调 ACK 前调用 `message.patch` / `feishu.updateCard()`，这会导致客户端闪烁或回滚。确需延时更新时先 ACK，再用回调 token 调 `/interactive/v1/card/update`。

## Runtime Operation Notes
- 从 Lodestar 自己承载的对话里执行 `systemctl --user restart feishu-daemon.service` / `lodestar-stop` / `restart` 这类会重启或停止当前 daemon 的命令时，工具调用显示 `aborted` 通常只是宿主进程被 SIGTERM 中断了，不代表操作失败。恢复后先用 `systemctl --user status feishu-daemon.service`、`journalctl --user -u feishu-daemon.service` 或 PID/日志确认结果，不要直接向用户汇报“重启未完成”。
- “测试当前改动”“发交互卡片到本群”“先看看效果”默认都属于**非授权**的 live-service 变更理由。除非用户在当前用户消息中明确要求，否则禁止执行任何会影响 `feishu-daemon.service` 或当前 live daemon 的操作，包括但不限于 `stop`、`restart`、`systemd-run` 起替代 daemon、手工 `bun daemon.ts` 接管、修改 service 指向、覆盖 live repo 代码后重启。
- live-service 操作授权不得跨 turn 生效：如果上一个 turn 同时要求“提交、重启”，后续新 turn 只说“提交”“继续”“再改一下”或提出新需求时，必须视为没有重启授权；需要重启时先停下来向用户要新的明确许可。

## Release Checklist
- 除非用户明确要求 minor 或 major 版本，否则只把 `package.json` 版本号按 patch 递增（`+0.0.1`）。不要根据变更范围自行推断 SemVer minor/major。
- 发布前用 `bun test` 和 `bun run build` 验证。
- 提交 release bump，创建 `vX.Y.Z` tag，push `main`，再 push tag。
- 用 `npm publish --access public` 发布 npm 包。
- 同一个版本也必须发布到 GitHub Packages。临时写入项目 `.npmrc`，内容包括 `@leviyuan:registry=https://npm.pkg.github.com` 和 `//npm.pkg.github.com/:_authToken=$GH_TOKEN`；运行 `npm publish --registry=https://npm.pkg.github.com --tag latest --access public` 后删除 `.npmrc`。不要跳过 GitHub Packages。
- 始终为 tag 创建对应的 GitHub Release。本机没有安装 `gh`；使用 GitHub REST API，从 `~/.git-credentials` 读取 token，并用 `jq -n --rawfile body /tmp/notes.md ...` 构造 JSON body。
- 写 release notes 前先读取最近的 GitHub Releases，并匹配现有风格。当前 notes 与 `CHANGELOG.md` 保持一致：使用中文、优先写“用户能感受到什么”，只保留必要的兼容提示；能用平铺短要点说清时不要再按实现拆成 `修复` / `改进`；非首个版本结尾保留 `**Full Changelog**: https://github.com/leviyuan/lodestar/compare/vA...vB`，首个版本改用源码快照链接。
- 只有用户在当前用户消息中明确要求时才重启正在运行的 user service；重启前先用 `systemctl --user list-units --all` 确认实际 unit，且该授权不得跨 turn 复用。

---
> Source: [leviyuan/lodestar](https://github.com/leviyuan/lodestar) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
