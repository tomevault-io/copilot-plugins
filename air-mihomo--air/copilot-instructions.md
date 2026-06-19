## air

> 本文档面向后续维护本项目的 AI 开发代理。开始编码前必须先阅读本文件，以及 `.codex/project_tasks_summary.md` 和与本次改动相关的 `.codex/*.md` 流程文档。历史拆分任务已归档为汇总文档；如果文档和源码不一致，以当前源码为准，并在相关 `.codex` 文档中补齐差异。

# AGENTS.md

本文档面向后续维护本项目的 AI 开发代理。开始编码前必须先阅读本文件，以及 `.codex/project_tasks_summary.md` 和与本次改动相关的 `.codex/*.md` 流程文档。历史拆分任务已归档为汇总文档；如果文档和源码不一致，以当前源码为准，并在相关 `.codex` 文档中补齐差异。

## 项目定位

`air` 是一个跨平台 mihomo 可视化管理器，目标是提供接近 Clash Verge / FlClash 的管理体验，同时保持 Rust 原生桌面应用的性能、低资源占用和清晰模块边界。

核心技术栈：

- Rust 2024 edition。
- GPUI 作为原生 GUI 框架，直接使用 Zed 官方源码仓库依赖。
- gpui-component 作为组件库，直接使用 longbridge 官方源码仓库依赖。
- mihomo 作为外部核心进程，通过配置文件、进程管理和 external-controller API 交互。
- QuickJS 通过 `rquickjs` 执行运行配置覆写脚本。

GUI 依赖策略：不要优先使用 crates.io 版本。`gpui` 和 `gpui_platform` 使用 Zed 官方源码仓库，`gpui-component` 使用 `longbridge/gpui-component` 官方源码仓库。首次接入或升级时必须拉取最新源码状态，并在 `Cargo.lock`、`.codex` 文档或提交说明中记录实际使用的 git commit，避免无法追踪的“最新 main”差异。若源码仓库 API 发生破坏性变化，以官方源码和示例为准调整项目代码。

当前锁定状态：

- `gpui` / `gpui_platform`：`https://github.com/zed-industries/zed`，`Cargo.lock` 解析到 `ee5c7b6d45faeccd40a285be63a853753c91eff0`。
- `gpui-component`：`https://github.com/longbridge/gpui-component.git`，`rev = "196b9259b562c26be97c92f88c798bbeefa9cb3d"`。
- `Cargo.toml` 中 `gpui` 未单独写 `rev`，是为了与 `gpui-component` 依赖的 Zed 来源保持一致，避免 Cargo 解析出两份 GPUI。

## 当前源码状态

项目功能已经基本完善。历史第 0 到第 7 阶段任务已汇总到 `.codex/project_tasks_summary.md`，`.codex/tasks/` 不再作为维护入口。

当前主路径具备：

- GPUI 主窗口、左侧导航、主题切换、状态栏、全局通知、托盘事件和关闭到托盘。
- 仪表盘聚合运行状态、内核生命周期按钮、运行时配置查看、日志/流量/内存监控。
- 订阅源管理、URL 导入、本地 YAML 导入、手动更新、定时到期更新、取消更新、排序、选择和缓存预览。
- 代理组离线配置展示、运行态选择回填、选择代理、单节点测速、组测速和 fixed 清理。
- 连接页 WebSocket 监控、HTTP 刷新、单连接关闭、筛选结果批量关闭和 Windows 进程图标缓存。
- 运行态规则页从 `/rules` 读取规则，支持过滤和通过 `/rules/disable` 临时启停规则。
- 设置页合并应用设置和 mihomo 常用配置编辑，GUI 设置保存到 `app.config.toml`，核心配置保存到 `core.common.config.yaml`。
- 覆写页编辑 `data/override.js`，在写出运行配置前通过 QuickJS 修改合并后的 `core.runtime.config.yaml`，并支持预览调试。
- app 命令路由、事件总线、快照仓储、后台 Tokio runtime、取消令牌和脱敏错误通知。
- Windows 下托盘、开机自启、UAC 提权 helper、内核服务安装/卸载/启动/停止，以及 TUN 场景优先通过内核服务托管核心。

仍需特别注意：

- 当前 app 主路径使用单配置文件：用户配置为 `config/core.common.config.yaml`，运行合并配置为 `config/core.runtime.config.yaml`。历史 `ProfileStore` 已不在当前源码中，`Profiles` 路由只作为配置编辑分支保留且不在侧边栏暴露；涉及多 profile 时必须先重新设计架构。
- `crates/air-config/src/merge.rs` 仍保留较完整的合并流水线，但当前启动主路径使用 `CoreConfigStore::merged_runtime_config()` 和 `AppServices::build_effective_runtime_config()`。
- `SaveConfig` 和启用订阅变更在核心运行时会重新写出 runtime 配置并调用 mihomo `PUT /configs` 重载；业务修改后的 YAML 仍由 `serde_yaml` 规范化输出，不能保留注释、锚点样式和原始排版。
- 订阅只接受可解析为 mihomo/Clash YAML 的内容；base64 节点订阅转换仍未实现，会返回预留诊断。
- 系统通知、导入导出/备份恢复、CI 打包发布、README、诊断导出文档仍未补齐。
- 平台能力主要覆盖 Windows。macOS/Linux 的 TUN 权限、自启、托盘、通知、服务化和打包发布仍为降级或 unsupported。
- `ConnectionsPageState::fake()`、`ConfigEditorPageState::fake()`、部分订阅样例仍作为测试或后端不可用时的降级夹具存在；生产业务入口不得重新依赖固定 fake 数据。

## 当前目录结构

继续实现时优先保持下列模块边界。允许按实际 API 细节调整，但不要把业务逻辑堆进 `main.rs` 或 GUI 组件中。

```text
crates/
  air-desktop/               # 最终 air binary、allocator、构建期 mihomo 下载和 Windows 图标资源
  air-app/                   # 应用装配、命令路由、事件、快照、后台任务和订阅控制
  air-ui/                    # GPUI 视图、组件、页面、路由、UI assets 和主窗口 Shell
  air-mihomo/                # mihomo 核心检测、external-controller API、领域模型、构建期发布包缓存、运行期资源释放与进程生命周期
  air-config/                # mihomo YAML 配置模型、解析、序列化、校验、覆写和合并
  air-storage/               # 目录规划、原子写入、核心配置、订阅缓存、覆写脚本和 app 设置仓储
  air-platform/              # 平台差异封装，业务代码不直接散落 cfg(target_os)
  air-settings/              # GUI 自身配置纯模型和 app.config.toml 兼容反序列化
  air-telemetry/             # tracing 初始化、日志保留、内存采样和敏感信息脱敏
  air-error/                 # 统一 AppError / AppResult
```

crate 依赖方向以 `docs/architecture.md` 为准。尤其注意：`air-app` 不依赖 `air-ui`，GUI 启动由 `air-desktop` 分流后调用；`air-config` 不依赖 `air-storage`；`air-mihomo` 不依赖 `air-storage`，订阅缓存读写通过 trait 注入。
UI 专用资源位于 `crates/air-ui/assets/`，其中 `icons/` 是项目 SVG 图标、`emoji/` 是 Twemoji SVG、`brand/` 是界面品牌图；Windows 可执行文件图标源位于 `crates/air-desktop/assets/app-icon.png`，由 `crates/air-desktop/build.rs` 转换为资源图标。

## 编码规范

- 所有代码文件均要使用UTF-8编码
- 所有新增 Rust 代码必须包含必要且具体的中文注释。注释解释设计意图、边界条件、平台差异和复杂逻辑，不写逐行复述。
- 模块必须小而清晰。公共 API 需要表达所有权、错误语义和线程/异步边界。
- 配置解析必须保留未知字段，避免破坏 mihomo 新版本或用户手写配置。
- 文件写入必须优先使用 `FileStore` 的原子写入和备份策略，不能直接覆盖用户配置。
- 任何涉及密钥、订阅 URL、认证头、代理密码、本地敏感路径的日志或 UI 通知必须脱敏。
- GUI 层只负责展示、局部交互状态和收集用户输入；配置合并、校验、进程管理、API 请求、订阅下载必须放在 app、mihomo、storage 或 platform 层。
- 跨平台逻辑必须通过 `platform` 模块隔离，不在业务代码中散落 `cfg(target_os)`。
- 新增依赖前必须说明用途，避免引入与 GPUI/gpui-component 重叠的 GUI 或状态管理框架。
- UI 优化继续遵守 `crates/air-ui/src/OPTIMIZATION_FOUNDATION.md`：复用 `crates/air-ui/src/components/`，使用 gpui-component 控件，保持 4px 栅格、8px 圆角上限、显式滚动条和统一通知入口。
- 清理 fake 数据时不要删除测试夹具；应改名为 `fake_for_test`、`sample_for_test` 或限制到 `#[cfg(test)]`，并让生产路径从 app snapshot、配置仓储、订阅缓存或 mihomo 事件构造页面状态。
- 所有关键流程必须要有详细的日志

## 常用命令

```powershell
cargo fmt
cargo check
cargo test
```

首次构建会由 `crates/air-mihomo/build.rs` 从 GitHub latest release 下载当前 target 对应的 mihomo 压缩包和 geodata，并缓存到 gitignore 忽略的 `mihomo/` 目录；源码仍通过 `include_bytes!` 读取这些本地压缩包，运行时解压到托管核心目录。`crates/air-desktop/build.rs` 只负责 Windows 可执行文件图标资源。当前支持 Windows/Linux/macOS 的 x86_64 和 aarch64 target；不支持的 target 会跳过核心包嵌入。如果引入 GUI、平台或打包相关依赖导致某平台暂时无法编译，必须在结果中明确记录目标平台、失败命令和原因。

## 文档规则

- `.codex/project_tasks_summary.md` 是历史任务汇总和当前遗留边界入口；不要恢复大量拆分任务文件，除非用户明确要求重新建立任务队列。
- `.codex/software_startup_flow.md`、`.codex/software_and_core_lifecycle.md`、`.codex/core_startup_flow.md`、`.codex/gui_core_interaction.md`、`.codex/data_flow_and_storage.md` 记录当前源码流程；修改相关链路时同步更新。
- `.codex/stage_0_3_review.md` 仅保留为历史阶段复盘，若源码后续继续偏离历史任务记录，以当前汇总文档为准。
- 修改任务或流程文档时保留历史事实和源码事实的边界，不要把“预留能力”写成“已完整实现”。

## mihomo 交互原则

- 通过 `docs/mihomo-api.md` 覆盖的接口实现运行态读取、代理切换、延迟测试、日志、流量、连接、规则和 provider 更新。
- 通过 `docs/config.yaml` 建模配置项，优先覆盖主流字段，同时保留扩展字段。
- external-controller 地址、secret 和 `SAFE_PATHS` 必须由配置或运行时装配注入，禁止硬编码。
- mihomo 进程必须可独立启动、停止、重启，并能处理启动失败、端口占用、配置无效和权限不足。
- 运行态 API 返回的临时状态不能直接写回用户 YAML；用户配置、订阅缓存、运行配置和 mihomo 当前运行状态必须在类型和注释上保持边界清晰。
- Windows TUN 场景优先保持 GUI 普通权限运行，只在核心进程或内核服务需要管理员权限时走 `platform::core_service` 或 `platform::elevated_process`；不要让 UI 回调直接调用 Win32 API。

## 测试要求

- 配置模型和 YAML 往返必须有单元测试。
- mihomo API 客户端必须有 mock server 或 trait mock 测试。
- 进程管理必须覆盖命令构建、状态机和失败路径，不要求单元测试真正启动 mihomo。
- GUI 状态转换至少覆盖纯状态 reducer / view model，不把核心行为只放进交互回调。
- app command router 的新增命令必须覆盖成功、失败、取消或核心未运行等关键路径，并验证错误脱敏。
- 任何修复 bug 的任务必须先补一个能失败的测试或说明无法自动化的原因。

---
> Source: [air-mihomo/air](https://github.com/air-mihomo/air) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-19 -->
