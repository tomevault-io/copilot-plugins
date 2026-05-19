## style

> - 页面的 CSS 样式你应该尽量通过以下两种方式实现：


# Style Rules

- 页面的 CSS 样式你应该尽量通过以下两种方式实现：
    1. 对于 `Ant Design` 或者 `@heroui/react` 等组件库提供的组件，请在组件库提供的 `ConfigProvider` 等类似的全局配置组件进行修改，如果你需要修改某个组件的全局样式，你可以在 `@/components/Registry.tsx` 中进行修改，它包裹了整个应用，如果你只需要单独修改某个位置的某个组件，请使用 `ConfigProvider` 包裹你需要修改的组件
    2. 对于一般样式，优先使用组件的 `className` 或者 `classNames` 或其他类名属性 + `tailwindcss` 实现
    3. 有且仅有以上两种方式无法实现时，请你使用 `style` 属性或者在 css 文件中定义样式

- 当你使用 `flex` 布局时，对于宽度或者高度需要保持固定的子元素设置 `flex-none`
- 对于 `React` 组件（也就是非 `div` 等 `html` 元素）的样式，请谨慎使用 `!important` 修改样式，请优先使用 `ConfigProvider` 或者组件暴露的属性（比如 `radius` / `shape`等）修改样式，最后再考虑使用 `!important`
- 请不要使用模板字符串的形式来实现动态样式，例如 ``className={`w-${width}px`}``，如果你想要实现条件类名，请使用 `deepsea-tools` 中导出的 `clsx` 函数，比如 `clsx("text-base", isPrimary ? "text-primary" : "text-secondary")`

---
> Source: [1adybug/docker-management](https://github.com/1adybug/docker-management) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-19 -->
