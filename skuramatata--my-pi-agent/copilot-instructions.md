## my-pi-agent

> 这是一个基于 Pi Agent 的个人开发助手扩展项目，核心目标是把记忆、任务编排、消息通道和 prompt 工程模块化。

# my-pi-agent — 项目上下文

## 项目定位
这是一个基于 Pi Agent 的个人开发助手扩展项目，核心目标是把记忆、任务编排、消息通道和 prompt 工程模块化。

`AGENTS.md` 只记录仓库级工程约定，不承载业务路由、人设风格或运行时 prompt。业务提示词、上下文拼装和策略描述统一放在 `packages/agent-runtime/src/prompt-engine/`。

## 关键目录
- `packages/agent-runtime/src/`：核心扩展，包含记忆、编排、任务队列、执行器和命令注册。
- `packages/agent-runtime/src/prompt-engine/`：prompt 模板、版本、上下文构建和统一渲染入口。
- `packages/channel-core/src/`：平台无关消息原语。
- `packages/channel-runtime-core/src/`：跨通道 request / coding / skill / card 共享运行层。
- `apps/feishu-channel/src/`：飞书 WebSocket 长连接通道。
- `apps/desktop/`：桌面端宿主预留目录，后续承接窗口、托盘、后台进程生命周期。
- `apps/web-console/`：Web 控制台前台预留目录，当前静态实现仍随 agent runtime 发布。
- `memory/`：本地记忆、工作上下文、任务队列和飞书会话上下文，默认不提交。
- `docs/`：规格、PRD、测试场景、决策记录和待办事项。

## 架构约定
- 自研 Orchestrator 是控制面，只负责分析、拆解、路由、DAG、队列、断点和重试。
- 顶层分层采用 `Router -> Strategy Workflows -> Patterns Library`：workflow 负责组合流程，pattern 只负责可复用执行 / 推理原语。
- 真正需要读文件、执行命令、编辑代码、验证结果的 agent step 交给 Pi 原生 `AgentSession.prompt()`。
- Pattern 是可复用执行 / 推理原语，不直接承担具体 agent 能力。
- 通用多步任务默认先落到 `dag` workflow；`sequential / parallel` 是 `dag` 内部的 preferred pattern，而不是顶层 workflow。
- `browser_use / swarm` 属于扩展 workflow；`reflection / tot` 已是受控 runnable reasoning patterns，只能在 catalog 明确开放的 workflow 中使用。
- DispatcherExecutor 根据 `step.kind` 或 action 前缀分派到 PiAgentExecutor / ShellExecutor。
- 子 Agent 默认不加载 extensions，避免飞书通道、调度器或命令重复注册。
- Prompt 不在业务代码里手写拼接，新增或修改 prompt 时先纳入 `prompt-engine` 并注册版本。
- 核心链路按职责拆分：runtime 只管控制流，state/support 只管持久化、lease、checkpoint 和恢复，task/protocol 文件只管协议 subtask 与 prompt，verification 继续走 harness registry，channel 继续走 `channel-runtime-core` 共享层。
- 新增或修改 `swarm` 能力时，优先按 `workflows/swarm-runtime.ts`（执行循环）、`swarm-runtime-state.ts`（状态/恢复）、`swarm-runtime-tasks.ts`（Lead/worker/checkpoint prompt）、`swarm-runtime-types.ts`（共享类型/小工具）分配职责，避免把状态、prompt 和调度重新混到一个文件里。

## 配置约定
- 密钥放 `.env`，不要放 `.pi/settings.json`。
- 飞书配置使用 `FEISHU_APP_ID` / `FEISHU_APP_SECRET`。
- 模型配置优先使用 `.env.example` 已声明的环境变量。
- `.pi/settings.json` 只放 Pi 项目级非密钥配置。
- `pi` CLI 默认视为“已全局安装”。在这个项目里不要反复向用户确认是否已安装 Pi。
- 当前机器上 `pi` 的全局可执行路径固定为 `$HOME/Library/pnpm/pi`；如果当前 shell 的 PATH 没带出来，就直接用这个路径执行。

## 开发约定
- 修改前先读现有实现，优先沿用当前接口、工厂模式和目录边界。
- 涉及编排、执行、prompt、记忆或消息通道的改动，需要同步更新 `docs/decision-log.md`。
- 改动 prompt 时同时确认 README 的 Prompt 工程说明是否仍然准确。
- 新增能力优先做成可替换组件，而不是把逻辑塞进入口文件。
- 发现单文件继续膨胀到 600 行以上时，先判断它是否混入了控制流、状态、prompt、IO 或渲染等多种职责；若是，优先做低语义风险拆分，再追加新能力。
- 改完至少运行 `git diff --check`；涉及 TypeScript 逻辑时再运行项目现有可用检查。

## Git 工作流
- 改完代码或文档后主动 commit，但不要自动 push。
- 只有用户明确说「push」时才执行 git push。
- commit message 使用 conventional commits 格式，说明文字用中文。
- push 前先 `git pull --rebase`，避免远端冲突。

---
> Source: [skuramatata/my-pi-agent](https://github.com/skuramatata/my-pi-agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-20 -->
