## hermes-slate-desk

> - React 19 + Vite 8 + Tailwind CSS 4 + Tauri 2

# Hermes Desktop Lite

## 项目栈
- React 19 + Vite 8 + Tailwind CSS 4 + Tauri 2
- shadcn/ui (zinc base, new-york style)
- Framer Motion, Sonner, Lucide React

## 代码规范
- JavaScript JSX，2空格缩进，无分号，双引号
- 函数式组件 + Hooks
- camelCase 变量/函数，PascalCase 组件
- 导入别名: `@` -> `./src`

## 状态管理
- React Hooks only (useState, useEffect, useCallback, useMemo, useRef)
- 无 Redux/Zustand

## 关键文件
- `src/App.jsx` - 主应用，状态管理
- `src/api.js` - API 调用
- `src/components/model-config-utils.js` - 模型配置工具
- `src/locales/` - i18n 翻译

## Agent skills

### Issue tracker
Issues live as markdown files under `.scratch/`. See `docs/agents/issue-tracker.md`.

### Triage labels
Default vocabulary (no mapping needed). See `docs/agents/triage-labels.md`.

### Domain docs
Single-context — one `CONTEXT.md` at repo root + `docs/adr/`. See `docs/agents/domain.md`.
- Hermes Guide: https://hermes.xaapi.ai/guide/introduction
- Hermes Skills: https://hermes-agent.nousresearch.com/docs/skills
- Admin Panel: http://127.0.0.1:9119/

---
> Source: [meespace/hermes-slate-desk](https://github.com/meespace/hermes-slate-desk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
