## echocode

> - 本文件适用于整个 `agentClient/`。

# AGENTS.md

## 适用范围与规则优先级

- 本文件适用于整个 `agentClient/`。
- 若更深层目录存在 `AGENTS.md`，以更深层规则为准。
- 用户当前任务中的明确要求优先于本文件的一般规则。
- 修改本文件、README 或其他项目入口文档前，必须取得用户允许。
- 注释和日志优先使用中文，保留必要的专业术语。
- 不得删除、篡改或迁移第三方许可证、NOTICE 和作者归属信息。
- 依赖版本、脚本名称和 workspace 配置以当前 `package.json`、`bun.lock` 与构建配置为准，本文件不重复维护版本快照。

## 架构红线与文件规模

### 适用范围

- 文件规模规则适用于 repo-tracked 业务源码中的 `.ts`、`.tsx`，主要覆盖 `apps/electron/src/` 和 `packages/*/src/`。
- 不包括 `*.test.*`、`*.spec.*`、`public/`、第三方静态资源和生成文件。

### 文件大小规则

- 单个 `.ts`、`.tsx` 文件不得超过 500 行，包含 import。
- 文件超过 400 行后，禁止继续增加新职责、新业务域或新功能块；需求必须进入该区域时，先拆分再实现。
- React 组件超过 300 行视为必须拆分候选，不得继续增加明显独立的新功能。
- 新增功能如果会突破上述限制，必须先完成拆分。
- `apps/electron/src/main/ipc.ts` 禁止直接增加新 handler；新增 IPC 必须拆到 `ipc-{domain}.ts` 或等价的域级文件。

### 存量超大文件

- 存量超大文件只允许同职责最小修复，不得借修复继续扩大职责。
- 新功能、跨域功能或明显新增职责必须先抽出 hook、service、子组件或域级 IPC 文件。
- 修改超过 400 行的文件前，必须先判断新逻辑能否放入独立模块。

## Monorepo 与模块边界

- `@app/shared`：共享类型、常量和无平台依赖的工具。
- `@app/core`：Provider、协议适配和核心能力。
- `@app/ui`：可复用 UI 组件。
- `@app/electron`：Electron 主进程、preload 和客户端界面。
- 共享包不得反向依赖 Electron 应用层。
- 主流程编排文件只负责生命周期、流程调度和跨服务协调；具体业务判断和策略逻辑放入独立 service。
- React 组件只负责展示与交互编排；非平凡业务逻辑必须放入 hook、atom 或 service。
- 状态所有权必须明确，跨组件和应用级状态统一使用 Jotai。
- 渲染层禁止新增直接调用 `window.electronAPI` 的位置；IPC 调用通过 atom、hook 或封装 service 暴露。
- 修改已有直接 IPC 调用链时，应收口而不是继续扩大调用范围。

## IPC 开发规范

新增或修改 IPC 时，必须同步检查：

1. 共享通道常量与请求、响应类型。
2. 主进程域级 handler。
3. Preload 的类型安全桥接。
4. 渲染层 atom、hook 或 service 封装。
5. 相应的边界与行为测试。

同时遵守：

- IPC 输入必须进行类型、路径、范围和权限边界校验。
- IPC、文件 IO、网络、持久化和用户可见失败统一使用 typed error。
- 新增 IPC 按业务域拆分，不得继续堆入总入口文件。
- 不得通过 IPC 暴露无边界的文件、命令或系统权限。

## 错误处理、异步任务与依赖分类

- 禁止空 `catch {}`。
- 只有清理、探测、幂等回收等 best-effort 分支可以忽略错误，并必须在旁边说明原因。
- 禁止只有 `console.error(error)`、`console.error(err)` 或 `.catch(console.error)` 这类无上下文日志。
- 错误日志至少包含模块名、动作名和错误对象。
- 仅局部断言、不可达分支和编程错误可以直接使用普通 `Error`。
- 新增定时器必须有明确 owner 和取消、清理路径。
- 禁止无归属的 fire-and-forget 任务控制主流程。
- `dependencies` 只放运行时真正 import 的包。
- 测试、构建、类型和脚本工具放 `devDependencies`。
- 平台 SDK 二进制放 `optionalDependencies`。
- 安装或升级依赖前必须查阅官方说明、兼容性和当前可用版本，不得只因为“最新版”就直接升级。

## TypeScript 与代码风格

- 禁止使用 `any`，应创建准确类型。
- 对象结构优先使用 `interface`。
- 仅类型导入优先使用 `import type`。
- 保持代码简单、直接、可读，避免过度抽象。
- 注释说明原因和约束，不重复描述代码行为。
- 新增代码必须遵守现有模块边界和命名风格，不得为了复用制造反向依赖。
- 使用项目现有 ESM、workspace 和 TypeScript 模块规则。
- 新增或修改行为优先按 Given、When、Then 组织测试。
- 修复缺陷时，应增加能够复现原问题的测试。

## UI 与状态管理

- 跨组件和应用级状态统一使用 Jotai；React 本地状态只用于组件内部的短生命周期状态。
- UI 优先复用现有组件和样式体系，通用组件优先放在共享 UI 层。
- 设置界面应兼容现有主题，并为主题扩展保留空间。
- 不得在 React 组件中混入复杂业务判断、文件 IO 或 IPC 编排。
- 使用 ShadcnUI、Radix 等组件体系时，遵守项目现有封装和可访问性模式。

## 常用命令

```bash
# 开发
bun run dev

# 构建与运行
bun run electron:start
bun run electron:build

# 类型检查
bun run typecheck

# 定向测试与全量测试
bun test <test-file>
bun run test
bun run test:fast

# 正式发版与测试包
./发布/release.sh
./发布/build-win.sh
./发布/build-mac.sh
./发布/build-win-dev.sh
```

- 正式支持平台为 Windows 和 macOS。
- 所有运行方式，包括开发模式，统一连接正式服务 `https://server.echocode.site`。
- 正式发版顺序固定为 `release.sh`、`build-win.sh`、`build-mac.sh`。

## 测试与提交规则

- 开发期间只运行相关定向测试。
- 禁止运行无文件参数的裸 `bun test`。
- 全量测试使用 `bun run test`。
- 只有在明确接受较高内存占用时才使用 `bun run test:fast`。
- 触及 IPC、主进程、状态管理、共享组件或会话链路时，仅在准备提交前统一运行一次全量测试。
- 全量测试通过后如果代码再次变化，测试结果视为过期。
- 全量测试通常运行时间较长；禁止设置明显不足的总超时。
- 工具超时不等于测试失败。需要时将测试放入后台，轮询真实退出码并保留完整日志。
- 不得因为工具超时而终止仍在正常运行的测试进程。
- 未经用户明确同意，不得运行会触发完整 Rust 编译链的命令，尤其是 `cargo test`。
- required 测试未通过时禁止提交。
- 不得在没有实际执行的情况下声称测试通过。

## Electron 打包约束

- `electron` 和 `@anthropic-ai/claude-agent-sdk` 必须在 esbuild 中标记为 external。
- Agent SDK 主包和目标平台子包必须进入安装包。
- 目标平台 SDK 子包必须在 `apps/electron/package.json` 中声明为 `optionalDependencies`。
- `electron-builder.yml` 必须包含 SDK 主包和目标平台子包。
- 普通运行时依赖原则上由 esbuild 打包；将普通依赖设为 external 时，必须确认其完整依赖树进入安装包。
- 不得在未验证模块解析路径的情况下改变 SDK binary 的打包位置。
- 修改打包配置后，必须检查目标平台依赖是否存在，并验证打包后的 Agent 功能。
- Windows 与 macOS 构建分别验证，不得用一个平台的成功结果替代另一个平台。

## SDK 环境变量安全

- 向 Agent SDK 子进程传递环境变量时，必须以清理后的系统环境为基础合并自定义变量。
- 必须保留 `PATH`、用户目录和 Shell 等必要运行环境。
- 必须剥离不应传给子进程的认证和渠道变量，不得破坏经明确配置的代理环境。
- 自定义 SDK 变量覆盖在清理后的环境之上。
- 修改环境变量构建逻辑时必须增加定向测试。
- 不得在日志、诊断信息或错误消息中输出 API Key、Token、完整认证头或其他凭证。

## 版本与正式发版

- 普通功能、修复、测试和重构不得修改 workspace package 版本。
- `@app/core`、`@app/shared` 和 `@app/ui` 是 private workspace 包，不因普通开发递增版本。
- `@app/electron` 版本只允许由 `发布/release.sh` 在正式发版时更新。
- 普通开发不得手工创建 `vX.Y.Z` 正式标签。
- 正式版本由发版脚本创建 release 提交、annotated 标签并原子推送。
- schema、迁移协议和第三方依赖版本仍按真实兼容性更新，不受 package 发版规则替代。
- 不得把普通代码提交和正式发版操作混为一体。

## 默认 Skills 版本契约

- 修改 `apps/electron/default-skills/<skill>/` 中任何内容时，必须同步递增对应 `SKILL.md` frontmatter 的 `version`，默认递增 patch。
- Skill 版本用于决定是否向老用户同步更新；版本不变时，老用户可能无法获得新内容。
- 新增 Skill 可以通过“目标不存在则注入”路径进入老工作区，不需要伪造旧版本。
- 修改 Skill 后必须验证首次安装和旧版本升级两条路径。
- Skill 自带的 README、references 和许可证属于运行资源，不得迁移到外层文档目录。

## 文档与开源边界

- README、LICENSE、NOTICE、第三方许可证和运行时 Skill 文档保留在客户端源码内。
- 临时设计、调研、交接、会议记录和内部发版手册统一放到仓库根级 `docs/agent/`。
- 不得在客户端源码目录新增内部设计文档。
- 源码中的 README 只记录开源用户和贡献者需要的信息。
- 内部文档不得被公共 README 或本文件引用。
- 移动文档时必须检查 Markdown 链接和代码中的路径引用。

---
> Source: [Echocode-Agent/Echocode](https://github.com/Echocode-Agent/Echocode) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-04 -->
