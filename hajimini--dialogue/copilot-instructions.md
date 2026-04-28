## dialogue

> 技术栈：Next.js 14 App Router + Supabase + Claude API

项目：AI拟真人设伴对话系统
技术栈：Next.js 14 App Router + Supabase + Claude API

代码规范：
- 使用 TypeScript，严格类型
- 使用 async/await，不用 .then
- API 路由使用 Next.js Route Handlers
- 数据库操作统一使用 Supabase JS SDK
- 组件使用函数式组件 + Hooks
- 样式使用 Tailwind CSS
- 文件命名：kebab-case
- 导出命名：PascalCase（组件），camelCase（函数/变量）

项目结构：
- 前端页面：src/app/(user)/ 和 src/app/(admin)/
- API路由：src/app/api/
- 核心逻辑：src/lib/
- 组件：src/components/

重要约定：
- 所有 Prompt 模板集中在 src/lib/ai/prompt-templates.ts
- 所有数据库类型定义在 src/lib/supabase/types.ts
- 环境变量用 process.env，不要硬编码任何密钥
- 错误处理：API 返回统一格式 { success, data, error }

---
> Source: [hajimini/Dialogue](https://github.com/hajimini/Dialogue) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-28 -->
