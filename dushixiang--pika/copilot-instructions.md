## pika

> - 不要直接使用 Tailwind CSS 工具类控制 Ant Design 组件的布局、尺寸、间距、溢出、颜色或内部子节点。Ant Design 组件的 `className` 不保证作用在期望的 DOM 节点，且组件样式优先级可能覆盖 Tailwind。

# Pika 开发约定

## Web 样式

- 不要直接使用 Tailwind CSS 工具类控制 Ant Design 组件的布局、尺寸、间距、溢出、颜色或内部子节点。Ant Design 组件的 `className` 不保证作用在期望的 DOM 节点，且组件样式优先级可能覆盖 Tailwind。
- 需要 Tailwind 布局或视觉样式时，在 Ant Design 组件外层包裹原生 `div`、`span` 等元素，并将 Tailwind `className` 放在这些原生元素上。
- 需要调整 Ant Design 组件本身时，优先使用组件自身的 props、`style`、`styles`、`classNames`、设计 Token，或 Ant Design 的 `Flex`、`Space`、`Row`、`Col` 等布局组件。
- 如果一个样式同时涉及 Ant Design 组件外部布局和组件内部外观，将两者分开：外部布局由包裹元素承担，组件内部外观由 Ant Design API 承担。

---
> Source: [dushixiang/pika](https://github.com/dushixiang/pika) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-16 -->
