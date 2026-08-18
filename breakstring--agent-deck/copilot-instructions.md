## agent-deck

> 本文件是 Agent Deck 项目的项目级 Coding Agent 指南。它补充全局规则，优先用于本仓库 `/Users/kenn/Projects/agent-deck` 内的开发、审查、测试和文档维护。

# AGENTS.md

本文件是 Agent Deck 项目的项目级 Coding Agent 指南。它补充全局规则，优先用于本仓库 `/Users/kenn/Projects/agent-deck` 内的开发、审查、测试和文档维护。

## 项目定位

Agent Deck 是一个本机 AI Agent 硬件控制台桥接项目。第一阶段目标是打通 `macOS + 妙联宝 N4 Pro + Codex` 的最快闭环：

- 采集 Codex 的运行状态、工具调用、审批请求和会话事件。
- 将状态归约成统一 `AgentState`，再生成硬件无关的 `LayoutPlan`。
- 在 N4 Pro 或 fake hardware surface 上展示 Agent 状态。
- 通过硬件输入选择 Agent、聚焦上下文，并在可控范围内响应 Codex `PermissionRequest`。

长期方向是支持更多 AI Agents 和更多妙联宝硬件。不要把实现写死成“Codex 专用按钮脚本”或“N4 Pro 专用渲染脚本”。项目核心边界应该保持为：

```text
Agent ingress -> NormalizedEvent -> AgentStateStore -> DeckMode/LayoutPlan -> HardwareSurface -> InteractionIntent/ActionExecutor
```

## 必读文档

开始非平凡开发前，先按任务范围阅读以下文档：

- `docs/superpowers/specs/2026-06-12-agent-deck-analysis.md`：总体需求、长期抽象、安全边界和非目标。
- `docs/superpowers/specs/2026-06-12-agent-deck-mvp-design.md`：第一版 `macOS + N4 Pro + Codex` 的 MVP 设计。
- `docs/references/agent-deck-roadmap.md`：长期 roadmap，后续任务拆分优先参考这里。
- `docs/references/stream-dock-scenes-research-2026-06-12.md`：妙联宝官方场景概念和 Agent Deck 内部 DeckMode 的边界。
- `docs/references/gemini-agent-deck-analysis-2026-06-12.txt`：Gemini 早期分析归档，只能作为参考，不可直接照搬其中伪代码。

如果需求与 roadmap 冲突，以用户最新要求为准，并同步更新 roadmap 或在交付说明中指出文档待更新。

## 工作流

- 默认中文回复，文档、注释、commit 信息也使用中文，除非用户明确要求英文。
- 修改前先看 `git status --short --branch`，确认是否有用户未提交改动。不要回退、覆盖或清理不是自己产生的改动。
- 需要隔离开发时使用 git worktree，目录放在 `.worktrees/` 下。该目录必须保持被 `.gitignore` 忽略。
- Python 依赖、命令和测试统一通过 `uv` 执行。不要手工维护裸 `pip` 流程作为项目默认路径。
- 常规子代理默认使用 `gpt-5.5`，推理强度 `medium`，除非用户明确指定。
- 交付前运行与改动范围匹配的测试。能跑全量时优先执行 `uv run pytest -q`。
- 不做破坏性 git 操作，例如 `git reset --hard`、`git checkout -- <file>`，除非用户明确要求。

## 当前代码结构

核心模块职责如下：

- `src/agent_deck/core/events.py`：统一事件模型、payload 脱敏、递归冻结、timezone-aware 时间校验。
- `src/agent_deck/core/state.py`：事件到 Agent 状态的内存归约。
- `src/agent_deck/core/decisions.py`：审批决策 broker、超时、默认 deny、跨线程/跨 event loop 等待。
- `src/agent_deck/core/modes.py`：DeckMode 和当前选择状态。
- `src/agent_deck/rendering/layout.py`：由状态和决策生成硬件无关布局计划。
- `src/agent_deck/hardware/fake.py`：无真实 I/O 的 fake hardware surface，用于测试和 daemon MVP。
- `src/agent_deck/hardware/streamdock_probe.py`：真实 StreamDock 设备只读诊断探针。
- `src/agent_deck/server/app.py`：本地 FastAPI daemon API，当前使用内存 runtime 和 fake hardware。
- `src/agent_deck/cli.py`：`agent-deckd`、`agent-deckctl`、`agent-deck-codex-hook` 三个 CLI 入口。
- `tests/`：当前核心行为的自动化测试。

新增模块时要维持分层，不要让 hardware driver 直接理解 Codex hook payload，也不要让 Codex adapter 直接操作硬件。

## 代码要求

新增或修改代码时，遵守以下要求：

- 每个新增或修改的代码文件顶部必须有符合语言习惯的文档级注释，说明文件职责、边界、副作用、关键依赖或约束。
- 每个新增或修改的函数、方法、类、协议、对外符号都必须有文档注释，至少说明语义、入参约束、返回值、错误处理和副作用。
- 文档注释默认写中文。已有英文 docstring 可以在触达时逐步改成中文或中英混合，但新增说明必须让中文读者能直接理解。
- 对外数据模型优先使用 Pydantic，涉及跨模块传递的状态快照优先保持不可变或清晰的复制语义。
- 所有时间字段必须使用 timezone-aware `datetime`。不要引入 naive datetime。
- 事件 payload 必须脱敏，不能在日志、错误或 API 输出中泄露 token、secret、authorization、api_key、password 等敏感值。
- 硬件输入不得直接执行 shell、AppleScript 或向 Agent 写入文本，必须先转换为业务 intent，再由 action 层执行。
- 高风险动作默认关闭或 fail-closed，尤其是审批、文本输入、未知前台窗口注入。

## 真实硬件边界

用户当前有真实妙联宝 N4 Pro，但代码必须始终保留 fake adapter，不能让真实硬件成为唯一测试路径。

处理真实设备时注意：

- 官方妙联宝 App 可能占用设备。需要真实接管设备前，先确认官方 App 是否已退出，或者通过 `doctor`/探针给出明确诊断。
- 诊断探针只能做只读 open/read/close。不要在诊断流程调用官方 SDK 的 `device.init()`，因为真实 SDK 的 `init()` 会唤醒屏幕、设置亮度、清空图标并刷新设备。
- `streamdock_probe.py` 当前设计为短暂 open，读取固件和序列号，然后 `close(notify=False)`。保持这个安全边界。
- macOS 上 PyPI `streamdock` 包可能加载 Linux `.so` 失败。真实探针优先支持通过 `AGENT_DECK_STREAMDOCK_SDK_PATH` 指向官方 `Python-SDK` 或 `Python-SDK/src`。
- N4 Pro 在断开通知或某些重启路径后可能只显示品牌图。真实 renderer 接管新枚举设备时必须先
  发送 `report_id=0 + HAN + zero padding` 的 1025 字节握手包，再执行常驻会话的 open/init；
  安全只读 probe 不得发送该握手。首次连接和每次重连都要执行，不能依赖官方 App 复位。
- macOS 设备权限不足时，vendored SDK 可能出现假成功：`device.open()` 为 True、`can_write`
  为 True、写入返回 0，但 `transport.get_last_error()` 实际是 `[HID] Device not open`，真机仍只
  显示品牌图。真实硬件调试不能只看这些返回值；握手 raw HID open/write 必须成功，并以真机
  显示变化作为 smoke 证据。从 Codex Desktop App 调试时应启用 Full Access 并完整重启 App；
  若仍报 `not permitted`，检查 macOS“隐私与安全性 -> 输入监控”并重新启动相关终端/App。
- 不要在自动化测试中访问真实 HID 设备。真实设备验证只能作为显式 smoke/manual 步骤。

当前真实 N4 Pro 安全探针命令示例：

```bash
AGENT_DECK_STREAMDOCK_SDK_PATH=/tmp/StreamDock-Device-SDK/Python-SDK uv run python - <<'PY'
from agent_deck.hardware.streamdock_probe import probe_streamdock_devices

for result in probe_streamdock_devices():
    print(result.model_dump())
PY
```

成功只读探针里 `can_open=True` 且 `can_init=False` 是预期结果，因为安全探针故意不调用 SDK `init()`。

## Codex 集成边界

当前项目已有三个 console scripts：

```bash
uv run agent-deckd
uv run agent-deckctl
uv run agent-deck-codex-hook
```

Codex 集成规则：

- `agent-deck-codex-hook notify` 是 best-effort。daemon 不可用时可以写 stderr，但应保持 exit 0，避免影响 Codex 正常流程。
- `agent-deck-codex-hook permission-request` 必须 fail-closed。daemon 不可用、响应非法或等待超时时，返回 deny 结构。
- hook helper 从 stdin 读取 JSON 时，空 stdin、非法 JSON、非 object 顶层应退出 2。
- 不默认采集完整用户 prompt。若未来要在触屏显示 prompt 摘要，必须显式配置并做脱敏。
- Codex Desktop App 与 Codex CLI 的运行面不同。做检测或安装器时不要假设用户当前一定在 CLI 中运行。

## 测试与验证

常用命令：

```bash
uv run pytest -q
uv run agent-deckctl version
uv run agent-deckd --port 88765
```

`agent-deckd --port 88765` 应在 Typer 参数层返回干净的范围错误，不应该出现 uvicorn/Python traceback。

本地 daemon smoke 示例：

```bash
uv run agent-deckd --host 127.0.0.1 --port 8765
uv run agent-deckctl simulate --session-id smoke --event-type session.started
uv run agent-deckctl status
printf '{"session_id":"smoke","tool_name":"shell","reason":"smoke"}' | uv run agent-deck-codex-hook permission-request --timeout-seconds 0.001
```

验证时不要留下后台 `agent-deckd` 进程。长时间运行命令结束前必须确认会话已停止。

## 文档维护

- 长期任务和模块拆分更新到 `docs/references/agent-deck-roadmap.md`，该文件不使用日期前缀，作为动态参考文档。
- 一次性分析、阶段设计或 implementation plan 可以放在 `docs/superpowers/specs/` 或 `docs/superpowers/plans/`，允许带日期。
- 外部分析材料归档到 `docs/references/`，并在使用时标明“可参考但需验证”的边界。
- 如果实现改变了官方场景、DeckMode、硬件能力模型、hook 安装方式或安全默认策略，必须同步更新相关文档。

## 交付口径

完成任务时，说明：

- 改了哪些文件和核心行为。
- 跑了哪些验证命令，结果是什么。
- 有哪些未完成风险或需要真实硬件/真实 Codex 配置继续验证的部分。
- 如果涉及 git 合并或发布，给出当前分支、commit、`git status --short --branch` 证据。

保持结论基于证据，不要用猜测替代命令输出、代码引用或测试结果。

---
> Source: [breakstring/agent-deck](https://github.com/breakstring/agent-deck) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-17 -->
