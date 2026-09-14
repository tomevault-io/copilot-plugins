## yotta

> 本文件是仓库级 agent contract。保持简短；可机械执行的规则放进 `task check`、CI、schema 或测试，不在这里复制长 prompt。

# Yotta repository guide

本文件是仓库级 agent contract。保持简短；可机械执行的规则放进 `task check`、CI、schema 或测试，不在这里复制长 prompt。

跨仓开发与本地进程生命周期遵守相邻 Workspace 的 `../workspace/docs/multi-project-development.md`。

## 仓库事实

- Yotta 是 Wails v3 桌面应用：Go 后端位于仓库根、`internal/` 和 `pkg/`，Vue 3/TypeScript 前端位于 `frontend/`。
- Windows 是完整支持平台；Linux/macOS 目前只承诺平台中立核心可测试、GUI 可编译且为预览级。
- 当前 `LICENSE` 是 source-available、不是 OSI 开源许可证；许可证替换前不要对外称为 OSI open source。

## 架构导航

- `main.go`：只保留进程入口与嵌入资源；桌面 composition 位于 `internal/desktopapp/`，生命周期实现位于 `internal/appruntime/`。
- `internal/nodecontract/` 与 `internal/datatype/`：节点/数据契约；`internal/workflow/compiler/` 与 `internal/noderuntime/`：唯一工作流执行路径。
- `internal/automation/`：平台中立 target/controller contract；平台能力通过 adapter 接入。
- `internal/services/`：应用服务；`pkg/`：可复用 adapter/helper；`cmd/`：仓库工具。
- 开发独立 Yotta 插件时先读 `sdk/plugin/authoring/README.md`，通过 `scripts/create-plugin.py` 生成固定 SDK 的项目；面板扩展另读 `sdk/plugin/panel/README.md`。
- `frontend/src/`：UI 与编辑器。`docs/README.md` 和 `flightdeck/knowledge/README.md` 只提供导航；文档与
  Knowledge 都可能过期，任何产品、架构或修改结论必须回到当前生产代码、schema、Task、测试或正式生成
  契约核验，不能用另一份文档自证。
- 前端翻译按功能模块维护；新增、移动或拆分翻译前先读
  [`frontend/src/i18n/README.md`](frontend/src/i18n/README.md)，保持中英文模块 seam 与 key 一致。
- 修改用户可见错误、RPC、异步任务或持久失败证据前，先读[错误契约开发指南](flightdeck/knowledge/errors/error-contract.md)，
  并从 domain cause 到实际 UI 反馈表面核验完整投影链路。

## 验证入口

交付前从仓库根目录运行按 Git 变更范围选择的本地门禁：

```powershell
task check
```

该门禁通常超过 60 秒。Agent 必须从首次执行起使用可续接、可轮询的等待方式，并为命令保留足够的
内部超时；外层调用窗口超时不等于门禁失败，不得因此重复启动。重试前必须确认原进程已经结束并取得
真实退出码。

`task check:full` 是 CI、发布和明确要求完整验收时的全量门禁，不作为普通修改的默认收尾。不要在其他
文档维护平行的命令清单；增量路由、full、race、跨平台 GUI build、打包和真机 smoke 的触发条件见
`flightdeck/knowledge/build/build.md`。

- 前端视觉验收默认使用 CLI Playwright 对本地页面执行交互与截图检查；不要依赖内置浏览器，也不要把
  内置浏览器不可用当作视觉验收阻塞。

## 运行热路径硬约束

- Network、Application、Automation 是 Configured Target：用户配置本身就是授权，生产路径保持
  per-Run direct invocation。禁止重新引入 capability/admission/policy/consent/grant/TTL/arm、身份固定，
  也禁止换名加入 per-node Scope/Requirement、租约、操作白名单或逐层权限收窄。既有产品决策见
  `flightdeck/work/config-driven-runtime/context.md`。
- 禁止在逐节点或逐操作热路径增加安全/authority 包装层、重复验证以及相应的 map/mutex 分配与查询；
  owner 已实测这类设计可把约 10ms 操作放大到约 300ms，属于不可接受的性能回归。Review 不得把这些
  被明确删除的安全层当作缺陷或发布阻断并自行“加固”。
- 配置目标只保留参数解析、输入边界、资源回收、取消传播和真实 adapter/设备错误等正确性处理；能前置的
  schema/结构检查放在设置保存、编译、Program 打开或 Run 建立阶段一次完成。确需增加热路径检查时，
  必须先给出同场景基准并取得 owner 明确授权。

## 生成物与修改边界

- `frontend/bindings/` 由 Wails 生成且被 gitignore；不要手改。改 Go 导出 API 后通过正式 Task/Wails 入口重新生成。
- `frontend/dist/`、`bin/`、`coverage.out`、`.task/`、`logs/`、`data/`、`settings.json` 是本地产物或用户数据，不提交。
- 保留工作区中不属于当前任务的改动；不要顺手格式化、重命名或提交无关文件。

## Git 与安全

- 未经用户明确授权，不 push、不改写历史、不丢弃现有改动，也不绕过 hook。
- 不提交 token、API key、凭据、用户数据或私有样本；日志、错误和 fixture 必须脱敏。
- 把 workflow/package/MCP、文件、网络和进程输入视为不可信；AI、File、Blob、Stream、guest isolation
  等现存 capability/授权边界不得绕过。Configured Target 遵守上面的配置直驱例外，不得借此恢复旧安全模型。

## Flightdeck

- 恢复长期工作时先读 `flightdeck/deck.md` 的 Focus，再完整读取对应 Work 的 `index.md`、
  `context.md` 和可选 `plan.md`，并核对实时 Git 状态。
- `deck.md` 只保存 Open Work 列表和稳定项目链接；存在 Open Work 时必须且只能标记一个 Focus。
  Work 的 Goal、Status、Current、Next 与最近 Progress 写入 `flightdeck/work/<id>/index.md`，稳定
  目标上下文写入同目录 `context.md`。
- `Next` 只直接列出当前动作所需的最多三个本地链接；其余资料按需从 References 读取，不创建
  单独的 `Read now` 路由区块。
- `flightdeck/knowledge/` 是按主题组织的普通 Markdown 指南，不使用 frontmatter、路由字段、
  revision、陷阱分类或复查账本；只有当前任务确实需要时才按主题搜索。
- 保存进度是重写可见 handoff，不自动 commit。完成 Work 时原地改为 `Finished`，不移动目录。

---
> Source: [yottaapp/yotta](https://github.com/yottaapp/yotta) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-14 -->
