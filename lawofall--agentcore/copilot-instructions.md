## color-tokens

> 配色 token 规范——写任何颜色（Tailwind 类或 CSS）时参考，只用语义 token、禁硬编码


# 配色 token 规范

只用语义 token，颜色由主题统一定义。完整 token 表与规格 → [`UI-Pattern索引.md` §配色 token](/docs/04-前端/UI-Pattern索引.md)。

## 单一定义源

OKLCH 语义色单源：**`packages/design-tokens/`**（`tokens.css` 桌面 / `mobile-light.css` 手机）。需要新颜色 → 先加进 `packages/design-tokens/tokens.css`，再视需要映射 Tailwind 类。

## 禁止

- **硬编码调色板与 hex**：`bg-blue-500`、`text-red-600`、`#3B82F6`、`bg-[#...]`。唯一彩色出口是语义 token。
- **`hsl(var(--…))` 包裹**：token 已是 OKLCH，`hsl()` 会产出非法值。
- **拿 `accent` 当成功色**：`accent` 是中性 hover 表面。成功一律 `success`。
- **绕过 token 新建散色变量**：先评估能否复用，必须新增则进 `packages/design-tokens/tokens.css`。

---
> Source: [Lawofall/AgentCore](https://github.com/Lawofall/AgentCore) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-09 -->
