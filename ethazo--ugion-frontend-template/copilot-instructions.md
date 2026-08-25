## ugion-frontend-template

> 前端模板项目。**动手前先读 `docs/`** —— 本项目有若干与常见做法不同的约定。

# CLAUDE.md

前端模板项目。**动手前先读 `docs/`** —— 本项目有若干与常见做法不同的约定。

完整文档索引见 `docs/README.md`。

## 技术栈

React + Vite + TypeScript ｜ pnpm

shadcn/ui + Tailwind v4 ｜ TanStack Router / Query / Table ｜ react-hook-form + zod ｜ axios ｜ Zustand ｜ dayjs

**不引入文档未列出的依赖。** 需要新依赖时先问。

## 分层与依赖方向

```
routes/     路由声明，必须薄
features/   业务，新项目整块替换
shared/     底座，跨项目复用
config/     按项目填写
```

| 规则 | 违反的后果 |
| --- | --- |
| `shared/` 不引用 `features/` | 业务删不掉 |
| `features/a` 不引用 `features/b` | 模块连片 |
| `shared/lib/` 不引用项目内代码 | 失去纯函数性质 |
| 路由文件不写业务逻辑 | 底座/业务分离失效 |

新文件放哪，按 `docs/02-architecture/directory.md` 的决策树。拿不准放 `features/`。

## 与常见做法不同的约定

最容易按习惯写错的部分：

| 本项目 | 常见做法 |
| --- | --- |
| 文件名 kebab-case（含组件） | PascalCase |
| 禁止默认导出（路由文件除外） | 组件默认导出 |
| queryOptions 工厂，命名 `xxxQuery` | 直接写 `useXxx` |
| 列表状态存 URL search params | 存组件 state |
| search schema 用 `.catch()` | 用 `.default()` |
| 类型由 `z.infer` 推导 | 手写 interface |
| 响应必须 zod 校验 | 直接用 `response.data` |
| 布局按路由绑定 | 按角色绑定 |
| 不预设全局菜单 | 一份全局菜单树 |
| 不用 mock、不写测试 | 引入 mock / 测试框架 |

## 核心约束

**鉴权**：HttpOnly Cookie，前端不持有凭证。会话四态（`unknown` / `authenticated` / `unauthenticated` / `expired`），**不能用布尔值** —— 混淆前两态会导致刷新时闪登录页。

**权限**：后端只给角色，前端配导航。路径前缀即权限边界。前端权限只是界面便利，**不是安全边界**。

**布局**：四套壳（admin / app / focus / blank），按路由绑定。FocusLayout 无导航。外边距由壳提供，页面不自设。

**平板**：约 1024px 需认真适配。侧栏收起、触摸目标 ≥40×40px、**任何功能不能仅靠悬停可达**。

## 开发顺序

业务模块固定从 schema 开始：

```
schema.ts → api.ts → queries.ts → columns.tsx → pages/ → routes/ → nav.ts
```

类型全部由 schema 推导，先写页面会导致类型手写、后期返工。

具体步骤见 `docs/07-ai/recipes.md`。

## 完成标准

```bash
pnpm typecheck
pnpm lint
```

**两者通过才算写完。** 不要在有类型错误的状态下报告完成。

改动涉及鉴权、列表、表单、布局、路由时，走 `docs/08-delivery/acceptance.md` 对应小节。

## 高频遗漏

按实际踩坑频率：

1. `withCredentials: true` 漏配 → 登录后立刻 401
2. 探测请求未排除在 401 拦截外 → 跳转死循环
3. 用布尔值表示登录态 → 刷新闪登录页
4. `?redirect=//evil.com` 未拦 → 开放重定向
5. `ROLE_ENTRY` 指向不存在的路由 → 登录后 404 且看不出原因
6. `config/nav.ts` 的 `to` 写错 → 点击 404
7. `loaderDeps` 未声明 → URL 参数变化不重新加载
8. loader 与页面用不同 key → 请求两次
9. 筛选变化不重置页码 → 空列表，用户误以为无数据
10. 提交时不禁用按钮 → 重复提交
11. 列定义写在组件内 → 表格反复重建
12. 平板上操作按钮仅悬停显示 → 功能不可达

## 不要做

| 禁止 | 原因 |
| --- | --- |
| 手改 `routeTree.gen.ts` | 自动生成，会被覆盖 |
| 手改 `shared/ui/` 下的 shadcn 组件 | 更新时覆盖；定制在 `shared/components/` 包一层 |
| 无参数 `invalidateQueries()` | 全应用缓存失效 |
| 手写字符串拼接路由路径 | 绕过类型检查 |
| 主动写测试、建 mock | 当前范围外 |
| 顺手重构或格式化无关代码 | 改动难以审查 |
| 不主动提交 | 用户明确要求时才提交 |

## 需要先问

引入新依赖 ｜ 改 `shared/` 公共约定 ｜ 后端格式与文档不符 ｜ 需要多角色 ｜ 需要跨域部署 ｜ 文档之间矛盾

其余按文档约定自行决定。

## 报告要求

**报告实际状态，不美化。** 类型检查未通过、某项未验证、用了变通方案、偏离了文档约定 —— 都要说明。

不说「已完成」而实际有类型错误。不说「应该可以了」—— 要么验证过，要么说明没验。

---
> Source: [ethazo/ugion-frontend-template](https://github.com/ethazo/ugion-frontend-template) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-23 -->
