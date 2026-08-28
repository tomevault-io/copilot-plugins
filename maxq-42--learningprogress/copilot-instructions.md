## learningprogress

> - 目标：记录已完成的学习成果，而不是 Todo List；数据和离线可用性优先。

# Delta Learning

## 项目约束

- 目标：记录已完成的学习成果，而不是 Todo List；数据和离线可用性优先。
- 技术栈：HTML、CSS、原生 JavaScript、localStorage、IndexedDB、Supabase REST、GitHub Pages PWA。
- 不引入 React/Vue、后端、服务器数据库替代方案或管理员级前端凭据。

## 关键目录

- `index.html`：三个页面视图与表单。
- `app.js`：本地记录、统计、UI、图片、导入导出、同步编排。
- `cloud-sync.js`：Supabase Auth/REST/Storage 客户端。
- `sync-core.js`：纯同步合并与软删除逻辑。
- `supabase/`：仅新增、可重复执行的 SQL migrations。
- `service-worker.js`、`manifest.json`：PWA 与离线资源。
- `tests/`：Node 级同步、云传输、Auth UI 约束测试；不发布到 Pages。

## 运行与测试

- Windows 本地启动：运行 `start-mobile-server.bat`，或以静态 HTTP 服务启动项目根目录。
- 核心检查：`node tests/sync-core.test.js`、`node tests/cloud-sync.test.js`、`node tests/auth-ui-contract.test.js`。
- 修改 PWA 静态资源后，递增 `service-worker.js` 的 `CACHE_NAME`，并验证首次联网和断网启动。

## 长期数据安全规则

1. 用户已有数据优先级最高；更新不得造成旧记录、设置或照片丢失。
2. 改表、字段、存储格式或同步逻辑前，先检查现有数据、调用方与迁移路径。
3. 数据库只使用向后兼容的增量 migration；禁止删表、删字段、reset、重建或覆盖生产数据。
4. 本地数据结构变更必须兼容旧记录；迁移先备份、转换后验证，失败时保留原始数据。
5. 保存和同步必须以 UUID、`updatedAt`、`deletedAt` 为准，不能覆盖其他设备的较新记录。
6. 高风险数据操作必须先说明风险并验证；不确定影响时先调查，不猜测执行。
7. UI 或重构不得改变既有数据含义，除非已有明确迁移方案。
8. 重要更新后验证旧记录可读取、保存、统计、导出和同步。

## 安全与发布

- 只允许浏览器公开 Project URL 和 Publishable key；禁止提交 Secret Key、`service_role` key、数据库密码或管理员凭据。
- 当前正式源码目录 `E:\LearningProgress` 没有 `.git`；发布时使用 `E:\LearningProgress_git_temp` 的现有 clone，或新建临时 clone。不要在正式目录执行 `git init`。
- 推送前检查 diff、敏感信息和 Service Worker；禁止 force push、改 remote、清空 localStorage 或删除 IndexedDB。

---
> Source: [MaxQ-42/LearningProgress](https://github.com/MaxQ-42/LearningProgress) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-28 -->
