## hermes-renderer-structure

> Hermes Renderer feature-sliced 结构与页面组件写法


# Hermes Renderer Structure（v1.3）

## 目录职责

| 目录 | 职责 |
|------|------|
| `panels/`、`components/` | 壳层装配（Sidebar、Shell、RightPanel）— 无业务 API |
| `pages/<Name>/` | 页面编排 + `components/` 子组件 |
| `features/<domain>/` | hooks、mapper、filter、校验 |
| `api/workApi.ts` | `window.hermesExperts` 封装 → `Work*` model |
| `api/hermesDefaultApi.ts` | 本地 default profile（Chat 等） |
| `model/` | 类型与状态枚举 |
| `registry/hermes-pages.tsx` | lazy 页面注册 |

## 新增页面

只改：`constants.ts` → `hermes-pages.tsx` → `pages/` → i18n。详见 `05-page-registry.md`。

## 页面 UI 模板

```tsx
<div className="hermes-page hermes-<name>-page">
  <header className="hermes-page__header">…</header>
  {/* error: hermes-page__error | empty: hermes-page__empty | loading: hermes-page__loading */}
</div>
```

## 禁止

- pages 内 `window.hermesExperts` / 原始 IPC 类型
- 新建全局 CSS 文件（扩展 `Hermes.css`）
- 在 Shell/Sidebar 写 summon / list 等业务逻辑

---
> Source: [loudon84/ai-os-desktop](https://github.com/loudon84/ai-os-desktop) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-18 -->
