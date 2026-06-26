## textile-app

> 本文档是 Textile 项目中 AI、Codex 和其他代码代理的开发约束。开始修改前先阅读本文档，再以当前代码为准确认实际行为。

# AGENTS.md

本文档是 Textile 项目中 AI、Codex 和其他代码代理的开发约束。开始修改前先阅读本文档，再以当前代码为准确认实际行为。

## 项目定位

Textile 是面向汉化组和翻译协作团队的本地优先工具。项目数据保存在用户本地文件夹或 `.hproj` 项目包中，不依赖业务服务器；协作以修改包导出、负责人合并和签名项目更新包为主。

## 当前技术栈

- Vue 3、TypeScript、Vite
- JSZip、vite-plugin-pwa
- Tauri 2、Tauri Updater
- File System Access API、IndexedDB、LocalStorage、Web Crypto API

不要无理由替换技术栈，不要引入大型 UI 框架。

## 架构规则

代码按以下职责分层：

- `src/pages/`：页面编排、交互状态、调用 service、展示结果。
- `src/components/`：可复用 UI 和局部交互。
- `src/services/`：业务规则、权限兜底、项目数据读写、导入导出。
- `src/services/projectFs.ts`：本地文件夹和内存 `.hproj` 根目录的底层读写抽象。
- `src/model/`：数据类型、状态规范化、权限 action 和默认角色权限。
- `src/utils/`：无业务状态的通用工具。

Vue 页面和组件不得直接读写项目文件。新增功能前先查找现有 service；需要项目文件访问时由 service 通过 `ProjectStorage` 读写，只有底层 adapter 或项目包打包能力才直接调用 `projectFs.ts`。不要为了一个功能一次性重写核心存储、修改包、权限或状态模块。

## 路由规则

当前项目不使用 `vue-router`，也不要引入 `vue-router`。

`src/App.vue` 负责：

- 解析 `window.location.pathname` 和 query。
- 使用 `history.pushState`、`history.replaceState` 导航。
- 监听 `popstate`。
- 持有当前项目、当前用户和项目摘要状态。
- 打开项目后调用各 service 的 `set*ProjectStorage()`；`set*ProjectRoot()` 仅作为旧调用兼容包装。

新增页面时：

1. 在 `App.vue` 的 `ProjectSection` 或顶层页面类型中加入明确路由。
2. 扩展 `parseRoute()`。
3. 使用现有 `navigate()`，不要在普通组件中自行操作 history。
4. 在 `ProjectLayout`、`ProjectSidebar` 和页面分发处同步接入。
5. 保证直接刷新和浏览器前进、后退行为可解释。

## 数据存储规则

项目常见结构：

```text
project.json
members.json
source/
entries/
terms/
tasks/
comments/
logs/
exports/
changes/
```

关键约束：

- `project.json` 是项目配置和文件索引。
- `members.json` 保存成员、公钥以及本地登录密码的哈希和盐。
- `entries/<file_id>/chunk_*.jsonl` 保存词条。
- `terms/terms.jsonl`、`tasks/tasks.jsonl`、`logs/events.jsonl` 是 JSONL。
- 评论按 `comments/<file_id>/<entry_index>.jsonl` 保存。
- 当前上下文主要保存在词条 `context` 字段，不要假设一定存在独立 `contexts/`。
- `.hproj` 打开后使用内存文件系统；修改后必须重新导出才会持久化。
- `exports/`、`changes/` 会随项目备份打包；普通导出使用可确认结果的保存对话框，不自动写入这些目录。签名信任过渡包会额外归档到 `changes/transitions/`。

修改数据格式必须保持向后兼容。新增字段优先设为可选并提供规范化默认值；不要静默改变既有字段含义。

## 权限规则

- 所有权限判断统一使用 `can(user, action, project?)`、`assertCan()` 或 `services/permissions.ts` 中的封装函数。
- UI 用权限控制按钮和入口，service 对关键写操作再次校验。
- 不要在页面或组件中散落 `roles.includes()`。
- owner/admin 特殊规则集中放在 `auth.ts`、`permissions.ts` 或权限模型中。
- `deny_permissions` 优先于角色权限和 `allow_permissions`。
- owner 的关键管理权限和防止最后一名管理员被锁死的检查不能绕过。
- 本地权限属于协作约束，不是服务器级安全边界；文档和 UI 不得把它描述为强隔离安全系统。

新增权限 action 时：

1. 在 `src/model/permissions.ts` 的 `PERMISSION_ACTIONS` 定义 action。
2. 加入对应权限组和需要的默认角色。
3. 在 service 写入口调用 `assertCan()` 或封装判断。
4. 在 UI 使用相同 action 控制入口。
5. 检查旧项目 `role_permissions` 的兼容策略。

## 状态与统计规则

- 词条正式状态只有 `untranslated`、`translated`、`proofread`、`reviewed`。
- 争议是独立的 `disputed` 标记，不要新增第五种主状态。
- 旧 `status: "disputed"` 和旧字符串 `proofread_by` 必须继续通过 `model/status.ts` 规范化。
- 保存者必须写入 `updated_by` 和 `updated_at`。
- 翻译、校对、审核审计字段通过现有状态函数和 `entries.saveEntry()` 维护。
- 手工保存导致译文或状态变化时，entries chunk 与 `logs/events.jsonl` 的版本事件必须通过同一写入计划提交；无变化保存不得产生空历史。
- 恢复历史译文必须追加新事件而不是改写旧日志，并将工作流重置为 untranslated/translated、清空校对审核字段、保留争议标记；恢复者只写 `updated_by`，不能被误记为历史译文的 `translated_by`。
- 所有页面、任务和导出摘要的进度统一走 `services/stats.ts`。
- 计算进度时必须传入项目的 `settings.progress_weights` 和 `settings.workflow`。
- 关闭审核后审核权重为 0，其余权重重新归一化；UI 显示“未启用审核”。

不要在页面中复制状态推断、校对次数或综合进度公式。

## 协作规则

不实现项目 Git 同步。普通 UI 不得出现项目 `pull`、`push`、`commit`、`branch`、`merge`、`HEAD`、`SHA`、`repository` 等 Git 协作术语。

正式协作路线：

- 成员导出普通修改包或任务修改包。
- 负责人导入、预览和处理冲突。
- 有权限的负责人发布签名项目更新包。
- 普通成员验证并接收项目更新包。
- `.hproj` 用于项目备份和迁移。

程序发布可以使用 GitHub Releases、下载页或 Tauri Updater；不要把程序版本分发和项目数据协作混为一谈。

## 修改包规则

- 修改包导入前必须完成 manifest、项目 ID、schema、内容哈希、签名状态、目标路径和冲突预检查。
- 预检查失败时不得写入项目文件。
- 项目更新包必须有有效签名、匹配的 `base_revision` 和 `target_revision`。
- 接收项目更新前要阻止覆盖未导出的个人修改。
- 项目更新包不得携带或覆盖成员本地密码哈希、密码盐和私钥。
- 维护修改包可能涉及项目设置、成员和凭据，必须明确提示并二次确认。
- 不要修改现有修改包格式，除非需求明确要求并提供向后兼容方案。
- 当前导入不是完整事务系统；任何扩展都要优先减少部分写入风险，不要假装已具备完整回滚。

## 数据安全规则

写入、导入、覆盖和删除前必须检查：

- 当前用户和权限。
- 项目 ID、目标记录和路径是否匹配。
- 先完成可失败的读取、解析和预检查。
- 更新索引文件前，正文文件是否已经成功写入或删除。
- 失败时是否保留原配置，是否需要清理中间文件并报告残留路径。
- 是否会覆盖本地密码、私钥、未导出修改或其他仅本机数据。

删除、导入、覆盖、撤销密钥、禁用成员、转让 owner 等操作必须二次确认。当前“移除项目”只清除最近项目记录和会话，不得递归删除磁盘项目文件夹。

## UI 规则

- 维持现有克制、工作台式界面，不做无关视觉重构。
- 使用项目已有组件、按钮、表单和颜色体系。
- 不引入大型 UI 框架。
- 权限不足时给出明确原因，不能只让按钮无响应。
- `.hproj` 内存修改、危险导入、身份私钥、项目移除等高风险状态必须有明确提示。
- 普通 UI 使用“协作与备份”“修改包”“冲突处理”“项目更新包”等术语。

## Service 规则

- service 的公开函数输入、输出和写入文件必须清楚。
- 关键写操作接受当前用户或从受控会话读取，并在 service 内做权限兜底。
- 不要让 service 依赖某个页面的临时状态。
- 项目存储通过现有 `set*ProjectStorage()` 或明确 `ProjectStorage` 参数传入；不要新增第二套全局项目存储机制。
- 结构化数据使用 JSON/JSONL 解析器，不使用脆弱的字符串拼接。
- 时间统一使用 ISO 字符串，优先调用 `utils/time.ts`。
- ID 沿用现有前缀加时间和随机片段的形式。

新增 service 时，应说明它属于哪个业务边界，并由页面调用 service，不要反向让 service 引用 Vue 页面。

## 代码风格

- Vue 使用 `<script setup lang="ts">` 和 Composition API。
- TypeScript 开启未使用变量和参数检查，新增代码必须通过 `vue-tsc`。
- 命名使用清楚的英文标识符；用户可见文案使用中文。
- 类型优先定义在 `model/types.ts` 或对应 service 的公开接口附近。
- 使用双引号和分号，保持现有格式。
- 注释只解释不明显的约束和风险，不写重复代码含义的注释。
- 不做无关格式化、批量重命名或顺手重构。

## Changelog 规则

- 用户可见功能、行为变化、兼容性变化、数据语义变化和重要修复必须同步更新根目录 `CHANGELOG.md`。
- 未发布修改写入 `[Unreleased]`，按 `Added`、`Changed`、`Deprecated`、`Removed`、`Fixed`、`Security` 分类；没有内容的分类不保留。
- 发布时将 `[Unreleased]` 内容归入与 `package.json` 一致的版本号和发布日期。
- 不记录纯内部重构、格式化、测试补充、构建哈希或其他不会影响用户和维护者判断的噪声。
- Changelog 描述已经实现的结果，不写计划、提交信息或未完成承诺。

## 文档规则

给 AI、Codex 和其他代码代理的开发约束集中写在本文档。其他文档除非确实需要描述用户安全、数据安全或真实技术行为，不写面向代理的流程约束、禁止事项或完成检查。

文档职责：

- `D:\documents\Textile_Project\vue-typescript-project-tutorial.md` 是长期稳定的 Vue 3 + TypeScript 学习教程，放在项目仓库外，不算 Textile 项目本体的一部分。
- `docs/TECHNICAL_DOCUMENTATION.md` 承接当前实现细节，包括目录结构、文件清单、模块职责、数据流、业务流程、导入导出、权限、统计、签名密钥、更新机制和维护注意事项。
- `docs/MANUAL.md` 是用户手册，面向实际使用者，描述“怎么用”和“遇到提示该怎么办”。
- `README.md` 是项目入口文档，只保留定位、快速开始、常用命令和文档导航。
- 根目录 `CHANGELOG.md` 只记录用户可见功能、行为、兼容性、数据语义和重要修复变化；纯内部整理或只影响学习材料的内容不写入。

程序修改后的文档更新优先级：

1. 用户操作、提示、流程或概念变化：优先更新 `docs/MANUAL.md`。
2. 代码结构、模块职责、数据格式、权限、统计、签名密钥、导入导出、更新机制变化：优先更新 `docs/TECHNICAL_DOCUMENTATION.md`。
3. 公开命令、依赖、启动方式、文档入口变化：更新 `README.md`。
4. 用户可见行为、兼容性、数据语义或重要修复变化：更新 `CHANGELOG.md` 的 `[Unreleased]`。
5. 只影响 Vue/TypeScript 通用学习方法且长期稳定：更新仓库外的 `D:\documents\Textile_Project\vue-typescript-project-tutorial.md`。

文档维护约束：

- 不把完整文件清单、函数细节、当前业务流程塞进长期教程。
- 不把零基础教程写进用户手册。
- 不在 README 中复制技术文档的大段内容。
- 新增文件、页面、service、数据格式或 Tauri 命令后，要同步更新技术文档对应章节。
- 如果无法确认某项实现，必须标注“需要进一步确认”，不要按预期架构编造。

## 推荐开发流程

1. 阅读 `AGENTS.md`、相关页面、service、model 和现有文档。
2. 用搜索确认调用方、数据路径、权限 action 和兼容逻辑。
3. 先描述最小变更边界和风险。
4. 优先修改业务 service，再接入页面。
5. 为删除、导入、覆盖和权限变更保留明确失败路径。
6. 检查 `.hproj`、普通文件夹、旧数据和无权限用户。
7. 运行最低构建检查，并按风险执行手动流程测试。
8. 完成后列出修改文件、行为变化、限制和测试方法。

## 常见任务

### 新增页面

扩展 `App.vue` 手写路由和页面分发；需要项目数据时调用已有 service。不要引入 `vue-router`。

### 新增项目设置

在 `ProjectConfig.settings` 增加向后兼容字段，提供默认规范化逻辑，由 service 保存 `project.json`，并检查修改包、`.hproj`、统计或导出是否需要同步支持。

### 新增导出格式

在 `model/types.ts` 扩展格式类型，在 `services/exporters/` 新增适配器，在 `exporter.ts` 接入，并同步设置页和导入导出页。检查键值、原文、隐藏词条、过滤和报告行为。

### 修改词条流程

同时检查 `model/status.ts`、`services/entries.ts`、`services/permissions.ts`、`services/stats.ts`、任务进度、文件进度、项目概览、统计页和成品导出。不要只改按钮文案。

### 修改修改包

同时检查导出收集范围、manifest、哈希、签名、预览、权限、冲突、预检查、普通导入、项目更新导入和旧包兼容。

### 修改程序更新

区分 Web/PWA `public/version.json` 和 Tauri Updater。Tauri 发布前必须通过 `npm run tauri:release:check`，私钥不得进入仓库。
更新状态优先级和用户可见文案统一通过 `services/appUpdatePresentation.ts` 计算；待刷新或待安装状态不得被后续“已是最新版”检查结果覆盖，页面和组件不要分别复制判断。

## 禁止事项

- 不引入 `vue-router`。
- 不实现项目 Git 同步。
- 不让 Vue 页面直接读写项目文件。
- 不把权限只放在 UI。
- 不在 UI 中散落角色字符串判断。
- 不绕过修改包验签、完整性和项目版本检查。
- 不递归删除用户的磁盘项目文件夹。
- 不把私钥、明文密码或发布签名密钥写入项目包或仓库。
- 不把未实现功能写成已实现。
- 不做无关重构或一次性重写核心模块。
- 不执行破坏性 Git、文件系统或环境配置命令。

## 完成检查

- [ ] 只修改了需求涉及的文件。
- [ ] 页面没有直接读写项目文件。
- [ ] service 写操作有权限兜底。
- [ ] 数据格式和旧项目保持兼容。
- [ ] 修改包、统计和工作流口径没有分叉。
- [ ] 高风险操作有明确确认和失败提示。
- [ ] 普通 UI 没有项目 Git 同步术语。
- [ ] 用户可见变化已更新 `CHANGELOG.md`。
- [ ] `npm run build` 通过；若未运行，明确说明原因。
- [ ] 按需求列出修改文件和手动测试步骤。

---
> Source: [Or-well/textile-app](https://github.com/Or-well/textile-app) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-26 -->
