## blog-admin

> - `src/pages/` 存放路由页面，`src/layouts/` 存放页面布局。

# Repository Guidelines

## 项目结构与模块组织

- `src/pages/` 存放路由页面，`src/layouts/` 存放页面布局。
- `src/components/` 存放可复用 Vue 组件，shadcn-vue 组件位于 `src/components/ui/`。
- Pinia store 放在 `src/stores/`，共享逻辑放在 `src/lib/` 或 `src/utils/`，组合式函数放在 `src/composables/`，插件放在 `src/plugins/`。
- 样式和前端资源放在 `src/assets/`，需要原样提供的静态文件放在 `public/`。测试应就近放在 `__tests__/` 中，并命名为 `*.test.ts`。`src/types/auto/` 下的文件由 Vite 插件生成，禁止手动修改。

## 构建、测试与开发命令

使用 Node `22.18+`（或 `24.12+`）和 pnpm：

```sh
pnpm install                 # 安装锁定的依赖
pnpm build                   # 使用 Vite 构建生产包
pnpm preview                 # 本地预览生产构建
pnpm type:check              # 执行 vue-tsc 类型检查
pnpm exec vitest run         # 执行全部单元测试
pnpm test:coverage           # 执行测试并检查 85% 覆盖率门槛
pnpm lint:fix                # 执行 ESLint 和格式化自动修复
```

禁止使用 `pnpm dev` 指令。

## 编码风格与命名约定

使用 Vue 3 Composition API 与 `<script setup lang="ts">`，采用两空格缩进、单引号和无分号风格，遵循 Antfu ESLint 配置。优先使用 `@/` 路径别名；组件使用 PascalCase，组合式函数使用 `useX`，store 使用 `useXStore`。已配置的 Vue、Pinia、VueUse、Vitest 和路由 API 会自动导入，仅类型导入使用 `import type`。关键公共 API、业务逻辑和不直观的决策必须添加简洁的 JSDoc 格式注释。

所有样式必须使用 Tailwind CSS；只有在确实需要传统 CSS（例如全局样式或 Tailwind 无法表达的规则）时才可使用 CSS/SCSS，并应说明原因。

开发或修改 UI 前，优先使用 shadcn-vue MCP 查询组件及用法，复用或组合已有组件，不得重复开发。只有确认没有合适组件后，才能编写自定义 UI；优先使用组件已有变体和语义化颜色。

开发任何组件或页面时，必须遵守 [ui-design-system.md](docs/ui-design-system.md) UI 规范文档。

## 交互安全：危险动作二次确认

对于不可逆或影响范围大的危险动作（如退出登录、删除、清空、强制下线、撤销发布、批量权限变更等），必须先弹出二次确认对话框（优先用 shadcn-vue `AlertDialog`）向用户说明动作与影响范围，用户明确确认后再执行，**不得一点即触发**。确认对话框须：清楚说明将发生什么与影响范围；危险动作用 destructive 变体；提供明确的取消路径（取消按钮、Esc、点遮罩）。

## TDD 与测试规范

开发 `src/components/`、`src/composables/`、`src/lib/`、`src/stores/` 或 `src/utils/` 下的文件时，必须遵循 Red-Green-Refactor：先编写能复现需求或问题的失败测试，再实现最小改动使测试通过，最后重构。Vitest 使用 `jsdom`，组件测试使用 Vue Test Utils；测试应覆盖公开行为和边界情况，环境变量使用 `vi.stubEnv`。覆盖率统计的目录中，行、语句、函数和分支覆盖率均须达到 85%。

## 安全与配置

复制 `.env.example` 为 `.env`，填写 `VITE_SUPABASE_URL` 和 `VITE_SUPABASE_PUBLISHABLE_KEY`。禁止提交 `.env` 或私密凭据；`VITE_*` 会暴露到客户端，只能使用可公开的密钥。

## Commit 与 Pull Request 规范

遵循历史中使用的 Conventional Commits，例如 `feat(supabase): configure frontend client`，主题应使用祈使语气并包含作用域。PR 必须说明变更和原因，列出验证命令；有对应问题时附上链接，UI 变更附截图或录屏，并明确说明配置、依赖或迁移变更。

## 交付前强制检查

所有任务完成后必须依次执行以下命令：

```sh
pnpm type:check
pnpm lint:fix
pnpm test:coverage
```

## Agent skills

### Issue tracker

本仓库的 issue 和规格说明以 Markdown 文件保存在 `.scratch/<feature>/` 下。详见 `docs/agents/issue-tracker.md`。

### Triage labels

使用默认 triage 标签：`needs-triage`、`needs-info`、`ready-for-agent`、`ready-for-human` 和 `wontfix`。详见 `docs/agents/triage-labels.md`。

### Domain docs

本仓库采用单上下文（single-context）布局，使用根目录 `CONTEXT.md` 和 `docs/adr/`。详见 `docs/agents/domain.md`。

---
> Source: [xcom1057136457/blog-admin](https://github.com/xcom1057136457/blog-admin) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-03 -->
