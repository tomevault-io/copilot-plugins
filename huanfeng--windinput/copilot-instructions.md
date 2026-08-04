## windinput

> <!-- Parent: ../../AGENTS.md -->

<!-- Parent: ../../AGENTS.md -->
<!-- Updated: 2026-06-29 -->

# wind-coordinator

## Purpose
输入法服务的"大脑"。实现 `wind_bridge::MessageHandler`，接收 C++ TSF 桥接层的全部事件（按键/焦点/IME 激活/光标/菜单），编排引擎、候选、UI、词库与持久化，维护完整输入状态机。上游是 TSF 桥接（`wind-bridge`），下游扇出到 `wind-engine`/`wind-candidate`/`wind-store`/`wind-ui` 等十余个 crate。

## Key Files
| File | Description |
|------|-------------|
| `src/lib.rs` | 模块导出（`Coordinator`/重启信号/设置 URL 提供者）；`is_foreground_fullscreen()` 全屏检测（供工具栏全屏隐藏） |
| `src/coordinator.rs` | 核心：`State`（全部输入态）/`ConfigBundle`/`Coordinator` 定义、构造器（`new`/`new_headless`）、`impl MessageHandler`、**按键主入口 `handle_key_event`（优先级链）**、候选翻页/高亮、统一导航 `apply_nav_key`、配置热重载 `reload_user_config` |
| `src/pipeline.rs` | `ModeKind`（单一活跃独占模式枚举）+ `Rewind`（夺取回退登记）；含与 Go 决策器的**刻意差异说明**（见下） |
| `src/handle_candidate.rs` | 候选生成/过滤/shadow/词频重排/分页/选词上屏/右键操作 |
| `src/handle_temp.rs` | 临时拼音 + 临时英文模式（触发判定/进出/候选刷新/上屏） |
| `src/handle_url.rs` | 网址模式（夺取缓冲 + 边界退格回退） |
| `src/handle_special.rs` | 引导键特殊模式（自带码表 + 全码上屏策略） |
| `src/handle_mode.rs` | 中英 / 简繁 / 方案 / 主题 / mix 融合模式切换 |
| `src/handle_punct.rs` | 标点编排 + 智能符号同键连按替换状态机（武装/触发/解除） |
| `src/handle_addword.rs` | 快捷加词 / 选词后自动造词 / `dict.add` |
| `src/handle_cmdbar.rs` | 命令直通车（cmdbar）集成：`init_cmdbar` + `EvalContext` 适配 + ime/dict 控制器 |
| `src/handle_menu.rs` | 主菜单 / 候选右键菜单分派、工具栏点击/刷新/位置持久化 |
| `src/handle_lifecycle.rs` | 配置重载、服务重启、独占模式进入/复位（IME 激活/焦点/composition 终止仍在 coordinator.rs 的 `impl MessageHandler`） |
| `src/handle_config.rs` | 配置更新处理（引擎/热键/UI/工具栏） |
| `src/handle_tooltip.rs` | 候选悬停提示（编码/拆字/拼音反查） |
| `src/hotkey_match.rs` | key_down 热键匹配 |
| `src/webdata.rs` | Web 设置数据 RPC（schema/dict/temp/freq/shadow/phrase/stats/theme 命名空间，经 wind-rpc 转发） |
| `src/stats.rs` | 输入统计采集 |
| `src/watchdog.rs` | 看门狗 |

> `src/handle_key.rs` 仅为模块占位（文档注释），实际按键路由在 `coordinator.rs::handle_key_event`。**注意：`keymap` 不在本 crate**，在 `wind-keys`（`use wind_keys::keymap`）；根 AGENTS.md 旧引用的 `wind-coordinator/src/keymap.rs` 已失效。

## For AI Agents

### Working In This Directory
- **按键唯一主入口 `handle_key_event`（coordinator.rs）**，优先级链顺序即正确性契约，改动前务必理解：key_up toggle 键切换 → 菜单转发 → key_down 热键 → 候选操作热键（Ctrl+数字）→ 加词模式 → 英文透传 → **夺取回退**（`VK_BACK` + `can_rewind`）→ **`state.active` 单点 match 分派** → 空缓冲模式激活 `try_activate_mode` → Ctrl/Alt 组合清空 → URL 夺取激活 → 以词定字 → `apply_nav_key` 统一导航 → 小键盘 → Esc/Back/Space/Enter/字母数字标点（engine_default）。新增逻辑须想清插在链的哪一环。
- **独占模式单点真相源**：临时拼音/临英/URL/特殊/mix 收敛为单字段 `State.active: Option<ModeKind>`（pipeline.rs），结构上保证「同一时刻至多一个独占模式」。新增模式 = 加一个 `ModeKind` 变体 + 一条 match 臂 + 一个 `handle_*_key`，**不要**再引入并行 bool。
- **不移植 Go 决策器**：Rust 各模式按 schema id 独立查引擎（`EngineManager::convert_with`），无被多模式改写的共享引擎，故 pipeline.rs 刻意不引入 Capability/Processor trait 抽象。读 Go 同名模块时勿照搬其 `decider`/`applyEngineDiff` 机制——此处不存在。
- **导航键走统一入口 `apply_nav_key`**（配置驱动 `keymap::NavKeys`，来自 wind-keys）：普通模式与所有候选模式共用；`include_printable` 区分码表型（`-`/`=` 作翻页）与文本/表达式型（临英/快捷，`-`/`=` 作输入）。禁止在各模式里各写一套翻页/高亮。
- **夺取回退（`pipeline::Rewind`）**：URL 抢前缀、z 抢前导拼音等「夺取式」模式登记快照后，退到前缀边界再退格 → 撤销夺取、把 `snapshot` 回放回正常码表输入流。URL 与 z 共用此机制，勿各写各的回退。
- **拼音逐步转换不变量**：`committed_text`/`committed_segs` 存「选中汉字累积、留组合区不上屏，全转完才整体上屏」；码表（五笔）选词消费整串、绝不进入此态。`preedit` 仅含输入码/拼音，**绝不含候选列表**。
- **配置热重载**：读配置统一经 `self.rt()`（`RwLock<Arc<ConfigBundle>>` 原子快照）；`reload_user_config` 整体替换 bundle，轻量项（标点/热键/候选数/导航键/配对）即时生效，重型项（引擎/方案/词典/字体）仍需重启。
- **锁与线程**：`State` 由单个 `Mutex` 保护，另有多个细粒度 `Mutex`/`Atomic`（pending_first_show、stat_recorded、fullscreen_cached 等）。cmdbar 动作经独立线程异步执行（`self_weak`），故控制器回调自锁的 coordinator 方法是安全的——切勿在持 `state` 锁时调用会再次取锁的方法。
- **工具栏显隐**对齐 Go 公式 `ime_active && toolbar_visible`（两者正交，见 `State` 注释），隐藏经 UI 层 50ms 防抖；全屏经 `fullscreen_cached` 后台异步刷新，勿在 bridge handler 线程同步调 `is_foreground_fullscreen`。

### Testing Requirements
- 本 crate **传递依赖 `windows` crate**（自身 `cfg(windows)` + 经 wind-bridge/wind-ui/wind-ipc 等），**不能在 host 跑 `cargo test -p wind-coordinator`**；需 Windows 目标交叉编译 / 设备验证。
- 纯逻辑函数（`detect_en_case`/`adapt_en_case`/`parse_pairs`/`punct_char` 等）逻辑独立，可借设备/CI 上的集成测试或无头构造器（`new_headless`）覆盖。

## Dependencies

### Internal
- `wind-ipc`（协议常量/键 hash）、`wind-bridge`（MessageHandler/KeyEventData/Push）、`wind-config`、`wind-store`（redb 持久化）、`wind-dict`、`wind-engine`、`wind-candidate`、`wind-transform`、`wind-theme`、`wind-ui`、`wind-cmdbar`、`wind-phrase`、`wind-keys`（keymap/VK/NavKeys）、`wind-quick-input`、`wind-reverse`、`wind-punct`

### External
- `tracing`、`anyhow`、`serde`/`serde_json`、`toml`、`chrono`、`fontdb`；`windows`（仅 `cfg(windows)`）。无 tokio（2026-07 移除，全 workspace 同步线程模型）

## 全局约束
按需引用根 `AGENTS.md`：VK 用 `keymap::VK_*`（来自 wind-keys，禁裸十六进制）；候选导航键走统一入口（`apply_nav_key`/`NavKeys`）；提交只用显式路径（禁 `git add -A`）；改完在 `wind_input/` 跑 `cargo fmt` 并与逻辑改动分开提交；日志 INFO 级不得含用户输入/候选/词库内容。

<!-- MANUAL: 此行以下为人工补充区，重新生成时保留 -->

---
> Source: [huanfeng/WindInput](https://github.com/huanfeng/WindInput) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-26 -->
