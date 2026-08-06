## t8-penguin-canvas

> 在修改代码、架构、配置、测试、UI、部署、GitHub 或技术文档前，完整阅读本文件、`SKILL.md`、`features.json`、`roadmap.md`、`package.json`、相关源码/测试，以及当前交接记录。项目没有 `meta.json`。

# T8-penguin-canvas 工作约束

在修改代码、架构、配置、测试、UI、部署、GitHub 或技术文档前，完整阅读本文件、`SKILL.md`、`features.json`、`roadmap.md`、`package.json`、相关源码/测试，以及当前交接记录。项目没有 `meta.json`。

## 当前权威开发路径

- 用户已明确决定恢复简单工作流：`E:\PenguinPravite\T8-penguin-canvas` 是唯一默认开发目录，当前分支为 `codex/vibex-workbench-node`。后续代码修改、开发运行、测试与本地提交都直接在这里进行。
- merge commit 的第一父提交是 core checkpoint `4e3061094014b5dc2720d52ed178a62e8469a9d3`，第二父提交是 release/F2 checkpoint `e0c6679b5a22539dd5b4983165ecc3f9d5c790e1`。
- `E:\PenguinPravite\T8-penguin-canvas` 的 `codex/vibex-workbench-node` 已无损采用双父语义 merge commit `68b5f72526a7272cc9787f6fda8b27a6f2fb54c8` 及后续修复；F2 与 core 的代码已经统一。
- `E:\PenguinPravite\T8-penguin-canvas-dev-integration-f2-core-20260720` 只保留为已完成集成的历史参考，不再作为必需开发入口，也不要在两个目录同时修改同一功能。
- `E:\PenguinPravite\T8-penguin-canvas-release-2.5.7` 继续冻结，不得在任何 release 命名路径开发。
- release/F2 的旧 `T8_ALLOW_LEGACY_F2_WORKTREE=1` 例外已随 HEAD 从 `9b6f6a4...` 移动到 checkpoint 而永久失效；这只用于继续阻止在旧 release 目录开发，不限制 canonical core。
- canonical core 的 `npm run worktree:core` 与 `npm run worktree:development` 都必须通过；`T8-penguin-canvas-dev-*` 仍可在特殊隔离任务中使用，但不是默认要求。release 路径必须拒绝 development。

## 永久保护

- 禁止 `reset`、`clean`、checkout 覆盖、整树复制、整树 ours/theirs 或任何丢弃本地修改的操作。
- 不得编辑或暂存源/core 工作树中的 `tools/ffmpeg-runtime/ffmpeg.exe`（143,314,432 bytes，SHA-256 `754A10CE2FC4A8C974FF492B351F58C02D35124D1D602FCF30F561FB1BD0F579`）。
- 不得编辑或暂存源/core 工作树中的 `tools/remove-ai-watermarks-runtime/README.md`（2,298 bytes，SHA-256 `04F13F0ADBB8593372FB9DDFA297A0DFB90D9EAD0325DE0CD340FCFE8B7CED56`）。
- 不得读取 retained/historical 项目数据库；数据库测试只能创建在系统临时目录并在测试后清理。
- 用户明确要求的日常开发可在 core 内修改、测试并按精确文件范围提交；版本升级、生产 build/打包、推送、tag 和 GitHub Release 仍须单独明确授权。未来正式包只构建一次。

## 已完成的无损集成

- 两边已分别制作显式 allowlist checkpoint；第三工作树完成 127 个冲突文件、1486 个冲突块的逐域语义合并，没有使用目录覆盖或整树 ours/theirs。
- 固定依赖顺序为 F2/F3 → B1 → B2/B3 → F4/F5 → F6 → F7 → Provider/媒体 → F9/F10/配置。
- 集成后的 193 个 TS 与 186 个 CJS 测试文件共 2872 项：2865 通过、7 个预期跳过、0 个遗留失败；type-check、public/rh-toolbox、writer/lifecycle、语法、JSON、worktree 和 diff 门通过。
- 集成树相对 release/F2 checkpoint 的产品语义仅增加 core 的 `nodemap.md`、`update.md`，以及记录集成事实的 `features.json` 更新。

## 当前剩余边界

- B1、F2-F7、schema32、全节点 RunEvent、B3 权限/安全清单、F9/F10 本地机制已经闭合；不得为了制造进度重复实现。
- 严格进度保持 27/32。B2/B3 仍缺真实历史端点、Windows 物理磁盘/安装升级回退、Provider 实网与资源负载证据。
- F8 必须由至少三个隔离客户端、两台真实 Windows 设备与 Electron 安装版完成；F9 必须使用真实域名、公共 DNS、证书、TLS/SNI 和反向代理；F10 必须完成真实公网红队与负载。
- 单机 localhost、mock、重复本地测试或手写汇总不能替代这些证据。没有对应环境时只维护失败关闭的采集/验证工具与事实记录，不勾选轮次。

## 防止再次跑错目录

- 新功能开始前先执行 `npm run worktree:check` 与 `npm run worktree:development`，并记录绝对路径、branch、HEAD、common dir。
- 默认 development 目录就是 `E:\PenguinPravite\T8-penguin-canvas`；`T8-penguin-canvas-dev-*` 仅在用户明确需要隔离任务时使用，release 目录只用于发布。
- 不通过复制目录同步代码，也不在 core 与历史 integration 目录同时修改同一功能。
- `predev`、`predev:vite`、`predev:backend`、`preelectron:dev` 必须保留 worktree role 门，新增开发入口也必须接入同一门。
- 任一保护文件漂移、未知 staged/unmerged、目录角色不符或外部证据缺失时，必须失败关闭并停止扩大结论。

---
> Source: [T8mars/T8-penguin-canvas](https://github.com/T8mars/T8-penguin-canvas) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-26 -->
