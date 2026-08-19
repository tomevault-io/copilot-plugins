## dsh-univer-office

> `dsh-univer-office` 是 DeepSeek Harness（DSH）的可安装 Cordis bundle。DSH 中一切能力都由插件组合；修改实现前阅读 [架构文档](docs/architecture.md)，修改用户能力或配置时同时阅读 [中文 README](README.zh-CN.md) 与 [English README](README.md)。

# AGENTS.md

`dsh-univer-office` 是 DeepSeek Harness（DSH）的可安装 Cordis bundle。DSH 中一切能力都由插件组合；修改实现前阅读 [架构文档](docs/architecture.md)，修改用户能力或配置时同时阅读 [中文 README](README.zh-CN.md) 与 [English README](README.md)。

## 开发命令

本项目要求 Node.js `>=22.19.0` 和 `pnpm@11.20.0`。使用 `pnpm install` 安装依赖，使用 `pnpm run typecheck` 检查全部 TypeScript compiler face。

`pnpm run build` 依次生成 Host/Client、Worker、Gateway、Render Machine 和 Viewer；也可运行 `build:lib`、`build:worker`、`build:gateway`、`build:render` 或 `build:viewer`。Smoke 测试消费生成的 `lib/` 与 `artifacts/`，必须先构建受影响目标。

按改动范围运行最窄的有效检查：Host/HTTP/Tools 用 `build:lib` 后运行 `test:host`；Client 用 `build:lib` 后运行 `test:client`；Skills 用 `test:skills`；Gateway、Worker、文件持久化、worktree 生命周期、Render Machine 或跨进程协议用对应构建后运行 `test:integration`。跨层或发布改动运行 `pnpm test`。仓库没有 lint 或 format 脚本。

## DSH 与 Cordis 约定

- 一个完整 capability seam 包含 Service Definition、Provider 和 Consumer。Service 是 Host 内部稳定入口；WebServer 与 Tools 保留 Consumer 职责，bundled Skills 保留 Provider 职责，不把 HTTP、工具 schema、UI 或具体 Gateway/Worker 实现泄漏进 Service。
- 函数插件命名导出 `name`、`inject`、`Config` 和 `apply` 中适用的成员，不添加 default export。依赖通过 `inject` 声明，不依赖手工挂载顺序。
- `cordis.patch.yml` 必须以裸包名挂载根插件，使 Loader 能从 `package.json` 的 `dsh.client` 清单发现浏览器 bundle；不要把组合入口改为相对路径或内部子路径。
- 注册是当前 Cordis fiber 拥有的可撤销 effect。使用 `ctx.effect()`、`ctx.on()` 或返回精确 disposer 的 registry API；卸载完成后不得遗留路由、工具、Skill provider、监听器、定时器或子进程。
- 可部署参数由已校验的 `Config` 提供；缺失或非法配置在最早可判断的位置失败。请求默认值在拥有该行为的 Provider 中通过显式 resolve 步骤产生，不在执行深处隐藏 `?? default`。
- 模型可见事实必须来自可回放的结构化会话事件。`univer_*` 工具返回恢复文件、worktree 和 Unit 目标所需的结构化值；Client 不解析自由文本或 bash 输出。工具 presentation 只依赖参数和持久结果。
- 工具 schema 和描述只使用模型完成任务所需的领域概念。新增工具时先确定执行、取消、超时、结构化输出、UI presentation、审批和 Skills 指引，并通过真实 Consumer 路径验证。

## 运行时与依赖边界

- 保持 [架构文档](docs/architecture.md) 中的依赖方向。WebServer 与 Tools 只调用 `ctx.univer`；Provider 才能组合 Gateway adapter、Unit Content adapter、缓存和 worktree 规则；Gateway Supervisor 只拥有进程生命周期。
- Client component 不直接 `fetch`；HTTP 集中在 `src/client/api/`，轮询和 mutation 集中在 hooks。Client 是会话日志与 Host 状态的投影，不拥有 worktree 真相，不读取本地文件或管理 Gateway。
- `src/shared/wire/` 只包含可 JSON 序列化的数据，不依赖 Node.js、React、Cordis 或 Univer。Host、Client、Gateway 和 Worker 的进程或 wire 边界需要运行时校验；同一进程内已由 TypeScript 保证的值不重复做敌意输入校验。
- 跨边界的 file、worktree 和 Unit ID 使用已有 branded 类型。所有文件、导入、导出和 SVG 资源都绑定显式 DSH session/tool workspace，并在 Provider 以 realpath 再次授权；共享 Gateway 进程不代表共享文件权限。
- Viewer URL 由 Host 基于已授权资源生成。Client 将文件、worktree、Unit、mode 和 scope 视为不透明身份参数，只能追加语言等纯展示参数。
- Gateway、Worker、Viewer 与 Render Machine 是独立 application。通过明确协议通信，不跨进程导入实现状态，不引入全局 `univer` CLI 或外部源码 checkout 依赖。

## 生命周期与安全不变量

- 内容写入只允许显式 `draft` worktree。`ready` 与 `reopen` 是正常 Agent 工作流；`merge` 与 `discard` 仅在用户明确要求且 DSH 审批通过后执行，不能作为自动收尾。
- `merged` 和 `discarded` 是终态，不可 reopen 或复用。Client 必须保留历史审阅投影，但移除终态 worktree 的修改操作和实时浮窗。
- 会话 ID 与 workspace scope 参与每次浏览器状态查询和 mutation 授权。缓存、轮询、窗口去重及审阅状态不得跨会话泄漏。
- 并发 Gateway 启动共享一个启动操作；失败、提前退出和卸载都释放状态并允许重试。只终止本插件拥有的进程，跳过被其他服务占用的端口，并验证健康端点身份。
- 取消与 teardown 必须达到静止状态：先阻止新的通知，再 abort/kill，并等待 Worker、浏览器或子进程退出。异步操作由一个明确 owner 管理 settlement、取消和清理，不留下孤儿任务。
- 当前插件不向模型提供截图。结构回读和 Slide lint 不能表述为逐像素视觉验收；视觉结果重要时明确说明仍需 Viewer 人工检查。

## TypeScript 与源码规则

- 使用 ESM、显式 `.ts` 相对导入和 `import type`。保持 `strict`、`noUncheckedIndexedAccess`、`exactOptionalPropertyTypes` 与 `verbatimModuleSyntax` 通过，不用宽泛断言或无说明的 `any` 绕过类型问题。
- Closed union 按 discriminant 穷尽处理；稳定错误由拥有该失败语义的层定义，HTTP 和工具 Consumer 映射错误，不解析错误字符串或暴露堆栈、内部绝对资源路径和原始子进程输出。
- 注释和 JSDoc 记录非显然的行为、失败、时序、所有权与安全用法，不复述代码或保留推理过程。空 `catch` 必须说明只吞掉哪一种预期失败。
- 延续所在文件的格式与命名，避免无关格式化。文件以一个换行结束，交付前运行 `git diff --check`。

## 测试与文档

- 测试描述行为并验证外部世界，不信任实现的自述。文件操作回读文件，生命周期测试检查最终状态与资源清理；新增回归覆盖成功、拒绝、取消及相关竞态。
- 测试使用临时 workspace 和动态端口，不依赖全局 CLI、已有演示文件、固定端口或本机遗留进程。只 mock 网络、时钟等昂贵或不确定边界，尽量通过真实 Service、Router、Gateway 或发布入口运行其余路径。
- Source plane 与 artifact plane 不混用：类型检查面向 `src/`；built smoke 明确消费刚生成的 `lib/`/`artifacts/`。不要让陈旧生成物掩盖源码或发布入口问题。
- 文档只描述当前状态，一项事实只保留一个归属位置：长期开发命令与 standing orders 属于本文件；架构、信任边界和设计决策属于 [架构文档](docs/architecture.md)；用户能力、配置和限制属于双语 README；具体模型工作流属于 `skills/`。
- 用户可见行为、配置或限制变化时同步更新两份 README；工具工作流变化时更新核心或对应 Unit Skill。不要在多处复制工具 schema、目录清单或实现细节。

## 生成物与工作区

- `lib/`、`artifacts/`、`dist/`、`*.tgz` 和 `*.zip` 是生成物，不手工编辑或提交。发布只包含从当前 `src/` 构建的产物；只有明确制作发布包时才运行会覆盖输出的 `scripts/build-dist.sh`。
- 精确锁定的 Univer SDK、API Reference 和 insiders 包是跨 application 的版本契约；升级时统一兼容版本族，并运行完整构建和集成测试。原生依赖必须在目标平台安装和验证，不能复制一个平台的 `node_modules` 作为通用产物。
- 开始工作先查看 `git status`，保留用户已有改动。搜索优先使用 `rg`/`rg --files`，只修改任务所需文件，不回退无关变化。

---
> Source: [dream-num/dsh-univer-office](https://github.com/dream-num/dsh-univer-office) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-19 -->
