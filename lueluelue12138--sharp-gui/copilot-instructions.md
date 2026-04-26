## sharp-gui

> > 本项目使用统一的 Agent 规范体系，请阅读入口文件获取完整规范：

# Sharp GUI — GitHub Copilot 项目指令

> 本项目使用统一的 Agent 规范体系，请阅读入口文件获取完整规范：
>
> 👉 **[../.antigravityrules](../.antigravityrules)**
>
> 该文件包含技术栈、全局红线、规范索引（Rules / Workflows / Skills）以及 Agent 检查清单。

## 关键约束速查（Copilot 内联补全适用）

- CSS Modules + CSS Variables，禁止 CSS-in-JS / Tailwind / Sass
- 命名导出 `export function`，禁止 `export default`
- `import type` 导入纯类型（`verbatimModuleSyntax` 已启用）
- 路径别名 `@/`，禁止 `../../../` 相对路径
- 原生 fetch（`api/client.ts` 封装），禁止 axios
- Zustand 单一扁平 Store，禁止 Redux
- 用户可见文本必须走 i18n（`en.json` + `zh.json` 同步）

---
> Source: [lueluelue12138/sharp-gui](https://github.com/lueluelue12138/sharp-gui) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-22 -->
