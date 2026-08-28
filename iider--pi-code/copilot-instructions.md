## pi-code

> Pi Code 用 WebUI 提供本地 Pi Agent 会话、工作区和模型配置界面。服务端把 Pi SDK 的

# AGENTS.md — Pi Code

## 项目是什么

Pi Code 用 WebUI 提供本地 Pi Agent 会话、工作区和模型配置界面。服务端把 Pi SDK 的
会话与模型能力适配为前端使用的 `/api/v1` 协议。

## 技术栈与架构

- `server/`：Fastify、TypeScript、Pi Agent SDK；提供 REST、WebSocket 和静态资源。
- `webapp/`：Vue 3、Vite；默认 WebUI 的可维护源码、测试和发布产物。
- `desktop/`：Tauri 桌面端封装，随包带入 `webapp/dist`。
- `docs/`：专题计划和迭代记录。

请求从 WebUI 进入 `server/src/routes.ts`，会话操作交给 `PiBridge`，模型配置交给
`ModelConfigurationService`；服务端通过 `/api/v1` 向前端提供稳定协议。

## 核心业务流程

1. `npm start -- --dangerous-bypass-auth --no-open` 启动本地服务。
2. WebUI 通过 REST 加载工作区、会话、供应商和模型，通过 WebSocket 接收会话事件。
3. 会话数据由 Pi SDK 读取；供应商表单保存到 `~/.pi/agent/models.json` 并刷新共享模型运行时。
4. `webapp/dist/` 是唯一随仓库发布的静态前端。

## 开发约定

- 细节为王，优雅至上：不做临时性、阅读困难的短视修改。
- 能抄不写，能连不造，能复用不原创：优先使用 Pi SDK、现有协议和稳定组件。
- 不覆盖工作区中的既有改动；提交前按最小功能单位拆分文件。
- 修改既有页面前先核对 `docs/` 中的视觉与行为基线；token 和测试通过不能代替真实页面复验。
- 服务端验证：在 `server/` 运行 `npm test` 和 `npm run typecheck`。
- Vue 前端验证：在 `webapp/` 运行 `npm test`、`npm run typecheck`、
  `npm run check:style -- --strict` 和 `npm run build`。

## 前端边界

- `webapp/` 是唯一前端源码；服务端和桌面端只发布 `webapp/dist`。
- API 类型集中在 `webapp/src/api/`，状态逻辑集中在 composable 和纯函数，页面不得自行复制协议映射。
- 通用视觉语义使用 `webapp/src/components/ui/` 和 `webapp/src/style.css`，不得为单页复制组件。
- 严格样式例外集中维护在 `webapp/scripts/check-style.mjs`，新增例外必须说明不可替代的产品约束。

## 已知问题

- 撤回消息后若未继续对话就重启服务，被撤回的尾部会因叶子指针不持久化而重新出现；
  继续发消息后永久正确。

## 收尾规范

每个最小功能单位完成后执行：自测、用户验收、最小提交，并按需更新 `AGENTS.md`、
README 或 `docs/` 中对应的长期文档，最后清场。用户验收通过前不提交。

---
> Source: [Iider/pi-code](https://github.com/Iider/pi-code) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-28 -->
