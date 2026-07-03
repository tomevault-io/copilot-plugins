## openclaudescience

> 本文件面向代码 Agent 和贡献者，只保留会影响正确性、安全性和维护边界的规则。长说明、背景分析和设计记录放到 `README.md` 或 `docs/`。

# InternAgents Agent Guide

本文件面向代码 Agent 和贡献者，只保留会影响正确性、安全性和维护边界的规则。长说明、背景分析和设计记录放到 `README.md` 或 `docs/`。

## 核心原则

1. DeepAgents 是外部 SDK。InternAgents 通过公开 API、backend adapter、middleware、tools 和资源配置扩展能力。
2. 不要修改或提交本地 `deepagents/` checkout。若必须使用稳定性不明确的 DeepAgents/LangGraph 接口，只能集中在一个 adapter/provider 文件里，并在该处说明升级风险。
3. 远程 Agent 服务和 SSH 计算资源是两类对象。前者是 UI 连接的已有 LangGraph/InternAgents 服务；后者是 local backend 通过 SSH 控制的 workspace。
4. 跨进程、跨机器、跨 SDK 的协议逻辑不要塞进页面组件，也不要堆进 `agent.py` 的大分支；放到 adapter、service 或 API route。
5. 配置和密钥默认本地化。仓库只提交安全默认值、示例和脱敏文档。

## 架构地图

- `agent.py`：LangGraph graph 组装和导出；导出 `agent`、`agent_local`、`agent_remote1` 到 `agent_remote8`。
- `internagent_resources.py`：读取 `internagent.resources.json` 或 `INTERNAGENT_RESOURCES_FILE`，生成资源配置。
- `ssh_backend.py`：把 DeepAgents backend protocol 映射到固定 SSH workspace。
- `kb_sync_middleware.py`：agent run 前后的 best-effort KB 同步。
- `thread_skill_middleware.py`：按 thread 选择和加载技能。
- `deepagent.config.json`：agent 行为配置，不承载机器私有信息。
- `ui/src/lib/remote-agent.ts`：浏览器侧 LangGraph SDK 封装。
- `ui/src/lib/config.ts`：浏览器可见的 Agent 服务配置。
- `ui/src/app/api/workspace/_lib/workspace.ts`：workspace/resource 文件访问，包括 SSH-backed 读取。
- `ui/src/app/connect/page.tsx`：Agent 服务连接页，不做 SSH 登录或远端安装。
- `ui/src/app/config/page.tsx`：local 模型、授权、主题配置，不做多服务器编排。

## 运行时边界

- local coordinator 使用 `langgraph.json`，资源任务应通过 `remote_url` 代理到独立 runtime。
- 资源 runtime 使用 `INTERNAGENT_PROCESS_ROLE=runtime` 和 `langgraph.runtime.json`，在具体 workspace 内创建真实 DeepAgent。
- `create_agent_for_resource(...)` 只有在显式设置 `INTERNAGENT_ALLOW_EMBEDDED_RESOURCE` 时，才允许把资源 backend 嵌入 coordinator。
- 只有当前 Agent 服务是 local 时，UI 才展示 local 管理的 SSH 资源。连接远程 Agent 服务后，该服务拥有自己的 workspace、tools 和 resources。

## Skill Catalogs

- Skill catalog 是 InternAgents 级共享能力，不只属于当前仓库。
- 默认搜索顺序：`~/.internagents/myskills`、`~/.internagents/imported-skills`、`skills`、`.internagents/imported-skills`。
- 本地或云端导入的技能默认复制到 `~/.internagents/imported-skills`，这样同一技能可以被多个 InternAgents 项目复用。
- `.internagents/imported-skills` 是项目内唯一允许作为预置技能分发的 `.internagents` 子目录；桌面打包会把这里的技能带进 runtime template。
- 当前项目只保存选中的 skill 列表和 `.internagents/active-skills` 下的 active symlink/copy；active skills、logs、pids、uploads、LangGraph state 仍是本地运行态，不提交。
- 技能配置写在 `deepagent.config.json` 的 `skills` 字段；改 catalog path、active path 或导入逻辑时，同步更新 Python、UI API 和配置页。

## 改动放置规则

- DeepAgents backend 能力：新增或修改 backend adapter，例如 `ssh_backend.py`。
- run 前后逻辑：放进 middleware，例如 `kb_sync_middleware.py`。
- 资源 schema 或选择逻辑：同步更新 `internagent_resources.py`、UI 资源读取、示例配置和文档。
- LangGraph graph 组装：`agent.py` 保持薄封装，不做安装、daemon 状态轮询或 SSH 协议管理。
- workspace/file 浏览：走 workspace API，不在 React 组件里直接访问本地文件或 SSH。
- 复用 UI 协议、状态和 helper：放到 `ui/src/lib`、`ui/src/app/types` 或组件附近的 `_lib`。

## DeepAgents 和 LangGraph

优先使用：

- `deepagents.create_deep_agent`
- `deepagents.backends.LocalShellBackend`
- `deepagents.backends.protocol` 中的 backend protocol 类型
- LangChain/LangGraph 公开的 middleware、message、graph、`RemoteGraph` 和 SDK API

避免：

- monkey patch DeepAgents 或 LangGraph runtime
- 依赖 SDK 私有路径、私有状态字段或未文档化 tool 名称
- 在有类型方法/protocol 可用时，把 SDK 对象当普通 dict 随意改写
- 在 UI、Next API route 和 Python backend 各写一套 SDK workaround

升级 `deepagents`、`langgraph` 或 `@langchain/langgraph-sdk` 时，先看 release notes 中的 backend protocol、tools、interrupt、stream mode、`RemoteGraph`、`LocalShellBackend` 变更。行为差异应封装成窄兼容层，不要到处加 fallback。

## SSH 和 Shell 安全

- Python shell 调用优先用 `subprocess.run([...], shell=False)`。
- 解析配置里的 SSH 命令时，用 `shlex.split(...)`，并集中在一个 backend/helper 层。
- TypeScript 优先用 `execFile`；确实需要 shell quoting 时，集中到一个 helper。
- 远端 payload 用 JSON、base64、argv 或其他结构化传输，不拼接未转义用户输入。
- normalize workspace 路径，并拒绝 `..`、绝对路径逃逸和 `~` 逃逸。
- workspace 浏览默认隐藏 `.env*`、私钥、runtime state、`node_modules`、缓存等本地产物。
- 远端命令必须有 timeout 和 max-output 限制。
- 不要新增第二套 SSH quoting、path sanitize 或 workspace traversal 逻辑。

## 配置和密钥

可以提交：

- `.env.example`
- `deepagent.config.json` 的安全默认值
- `internagent.resources.json` 的 local 默认资源
- `internagent.resources.example.json` 等示例
- 脱敏后的 `.internagents/imported-skills` 预置技能
- `ui/deepagent-ui.config.json` 的安全默认 UI 配置
- README/docs 中的脱敏示例

不要提交：

- `.env` 或 `ui/.env*`
- `internagent.resources.local.json`
- 真实服务器 IP、用户名、SSH alias、私钥路径、隧道端口或 API key
- `.internagents/` 下除 `imported-skills` 外的运行态、`.langgraph_api/`、日志、pid、临时 active skills 或 `node_modules`
- 本地 `deepagents/` checkout

配置 schema 变化时，同步更新 Python 解析、UI 读写、示例配置、README/docs，以及本文件中的相关规则。

## 验证

按改动范围运行对应检查。

文档或配置：

```bash
git diff --check
python3 -m json.tool deepagent.config.json >/dev/null
python3 -m json.tool internagent.resources.json >/dev/null
python3 -m json.tool ui/deepagent-ui.config.json >/dev/null
```

Python agent/backend：

```bash
.venv/bin/python -m compileall agent.py internagent_resources.py ssh_backend.py kb_sync_middleware.py thread_skill_middleware.py
.venv/bin/python -c "import agent; print(agent.MODEL)"
```

Shell 脚本：

```bash
bash -n scripts/dev.sh
```

UI：

```bash
npm --prefix ui run lint
npm --prefix ui run build
```

远程资源改动还要覆盖：SSH 不可达、缺少 `ssh_command` 或 workspace、workspace 逃逸被拒绝、短命令执行、大输出截断、timeout 生效、密钥和 `.env` 不通过文件浏览暴露。

## 收尾检查

- 看 `git status --short`，不要回滚用户已有改动。
- 不要把 SDK 升级、SSH backend、UI 交互、配置 schema 和纯文档改动混在一起，除非它们确实互相依赖。
- 如果同一个远程/资源概念在 UI、Next API、Python 中出现三套 schema，页面组件开始拼 SSH 命令，或 `agent.py` 开始承载安装、daemon、进程管理逻辑，先重构边界再继续。

---
> Source: [qzzqzzb/OpenClaudeScience](https://github.com/qzzqzzb/OpenClaudeScience) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-03 -->
