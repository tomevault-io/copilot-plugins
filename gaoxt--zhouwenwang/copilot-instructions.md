## tailwind-headless-constraints

> **必须严格遵守以下规则，禁止使用任何不存在的 className，避免 AI 幻觉生成虚假类名。**

# Tailwind CSS + Headless UI 严格约束规则

**必须严格遵守以下规则，禁止使用任何不存在的 className，避免 AI 幻觉生成虚假类名。**

## ⚠️ 核心原则

- **只使用官方文档确认存在的类名**
- **禁止使用小数点数值**（如 `gap-2.5`、`p-1.5` 等）
- **禁止自创组合类名**
- **所有类名必须经过验证**

## 🎯 Tailwind CSS 间距系统（Spacing Scale）

### ✅ 允许的间距值（Tailwind v4.1）

**基础间距单位：`--spacing: 0.25rem` (4px)**

```
0     = 0px
px    = 1px
0.5   = 2px     (0.125rem)
1     = 4px     (0.25rem)
1.5   = 6px     (0.375rem)
2     = 8px     (0.5rem)
2.5   = 10px    (0.625rem)
3     = 12px    (0.75rem)
3.5   = 14px    (0.875rem)
4     = 16px    (1rem)
5     = 20px    (1.25rem)
6     = 24px    (1.5rem)
7     = 28px    (1.75rem)
8     = 32px    (2rem)
9     = 36px    (2.25rem)
10    = 40px    (2.5rem)
11    = 44px    (2.75rem)
12    = 48px    (3rem)
14    = 56px    (3.5rem)
16    = 64px    (4rem)
20    = 80px    (5rem)
24    = 96px    (6rem)
28    = 112px   (7rem)
32    = 128px   (8rem)
36    = 144px   (9rem)
40    = 160px   (10rem)
44    = 176px   (11rem)
48    = 192px   (12rem)
52    = 208px   (13rem)
56    = 224px   (14rem)
60    = 240px   (15rem)
64    = 256px   (16rem)
72    = 288px   (18rem)
80    = 320px   (20rem)
96    = 384px   (24rem)
```

### ❌ 禁止使用的间距值

```
// 🚫 错误 - 不存在的小数点间距
gap-2.5
p-1.5
m-3.5
space-x-4.5

// 🚫 错误 - 不存在的数值
gap-15
p-17
m-13
```

### ✅ 正确的间距类名

```jsx
// ✅ 正确 - 使用存在的间距值
<div className="gap-2 p-4 m-3 space-x-6">
<div className="px-4 py-2 mx-auto">
<div className="mt-8 mb-4 ml-2 mr-6">
```

## 📏 Gap 系统规则

### ✅ 允许的 Gap 类名

```css
gap-0, gap-px, gap-0.5, gap-1, gap-1.5, gap-2, gap-2.5, gap-3, gap-3.5, gap-4, gap-5, gap-6, gap-7, gap-8, gap-9, gap-10, gap-11, gap-12, gap-14, gap-16, gap-20, gap-24, gap-28, gap-32, gap-36, gap-40, gap-44, gap-48, gap-52, gap-56, gap-60, gap-64, gap-72, gap-80, gap-96

gap-x-0, gap-x-px, gap-x-0.5... (同上数值)
gap-y-0, gap-y-px, gap-y-0.5... (同上数值)
```

### ❌ 禁止的 Gap 用法

```jsx
// 🚫 错误 - 不存在
<div className="gap-2.5"> // Tailwind 中不存在 gap-2.5
<div className="gap-15">  // 不存在 gap-15
<div className="gap-13">  // 不存在 gap-13
```

### ✅ 正确的 Gap 用法

```jsx
// ✅ 正确
<div className="flex gap-4">
<div className="grid grid-cols-3 gap-6">
<div className="flex gap-x-4 gap-y-8">
```

## 🎨 Tailwind CSS 颜色系统

### ✅ 标准颜色等级

**每种颜色都有以下等级：**
```
50, 100, 200, 300, 400, 500, 600, 700, 800, 900, 950
```

### ✅ 允许的颜色类名

```css
/* 基础颜色 */
black, white

/* 灰色系 */
slate-50, slate-100, ..., slate-950
gray-50, gray-100, ..., gray-950
zinc-50, zinc-100, ..., zinc-950
neutral-50, neutral-100, ..., neutral-950
stone-50, stone-100, ..., stone-950

/* 彩色系 */
red-50, red-100, ..., red-950
orange-50, orange-100, ..., orange-950
amber-50, amber-100, ..., amber-950
yellow-50, yellow-100, ..., yellow-950
lime-50, lime-100, ..., lime-950
green-50, green-100, ..., green-950
emerald-50, emerald-100, ..., emerald-950
teal-50, teal-100, ..., teal-950
cyan-50, cyan-100, ..., cyan-950
sky-50, sky-100, ..., sky-950
blue-50, blue-100, ..., blue-950
indigo-50, indigo-100, ..., indigo-950
violet-50, violet-100, ..., violet-950
purple-50, purple-100, ..., purple-950
fuchsia-50, fuchsia-100, ..., fuchsia-950
pink-50, pink-100, ..., pink-950
rose-50, rose-100, ..., rose-950
```

### ❌ 禁止的颜色用法

```jsx
// 🚫 错误 - 不存在的颜色等级
<div className="bg-blue-150">   // 不存在
<div className="text-red-75">   // 不存在
<div className="border-green-550"> // 不存在
```

## 🔤 字体大小系统

### ✅ 允许的字体大小类名

```css
text-xs     = 0.75rem
text-sm     = 0.875rem
text-base   = 1rem
text-lg     = 1.125rem
text-xl     = 1.25rem
text-2xl    = 1.5rem
text-3xl    = 1.875rem
text-4xl    = 2.25rem
text-5xl    = 3rem
text-6xl    = 3.75rem
text-7xl    = 4.5rem
text-8xl    = 6rem
text-9xl    = 8rem
```

### ❌ 禁止的字体大小

```jsx
// 🚫 错误 - 不存在
<h1 className="text-xxl">    // 不存在
<p className="text-medium">  // 不存在
<span className="text-1xl">  // 不存在
```

## 🏗️ Headless UI 组件约束

### ✅ React Headless UI 组件

**@headlessui/react** 包含以下组件：

```jsx
// ✅ 正确的 Headless UI 组件
import {
  Dialog,
  Disclosure,
  Menu,
  Popover,
  RadioGroup,
  Switch,
  Tab,
  Transition,
  Listbox,
  Combobox
} from '@headlessui/react'
```

### ✅ 正确的 Headless UI 用法

```jsx
// ✅ Dialog 组件
<Dialog open={isOpen} onClose={setIsOpen}>
  <Dialog.Panel className="bg-white p-6 rounded-lg">
    <Dialog.Title className="text-lg font-medium">
      Title
    </Dialog.Title>
    <Dialog.Description className="text-gray-500">
      Description
    </Dialog.Description>
  </Dialog.Panel>
</Dialog>

// ✅ Menu 组件
<Menu as="div" className="relative">
  <Menu.Button className="px-4 py-2 bg-blue-500 text-white rounded">
    Options
  </Menu.Button>
  <Menu.Items className="absolute mt-2 bg-white border rounded shadow-lg">
    <Menu.Item>
      {({ active }) => (
        <a className={`block px-4 py-2 ${active ? 'bg-gray-100' : ''}`}>
          Item 1
        </a>
      )}
    </Menu.Item>
  </Menu.Items>
</Menu>
```

### ❌ 禁止的组件使用

```jsx
// 🚫 错误 - 不存在的组件
import { Modal } from '@headlessui/react'     // 应该用 Dialog
import { Dropdown } from '@headlessui/react' // 应该用 Menu
import { Tooltip } from '@headlessui/react'  // Headless UI 没有 Tooltip
```

## 📱 响应式设计约束

### ✅ 标准断点

```css
sm:   = @media (min-width: 640px)
md:   = @media (min-width: 768px)
lg:   = @media (min-width: 1024px)
xl:   = @media (min-width: 1280px)
2xl:  = @media (min-width: 1536px)
```

### ✅ 正确的响应式用法

```jsx
<div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-4">
<div className="p-4 md:p-6 lg:p-8">
<div className="text-sm md:text-base lg:text-lg">
```

## 🎯 布局系统约束

### ✅ Grid 系统

```css
/* Grid 列数 */
grid-cols-1, grid-cols-2, grid-cols-3, grid-cols-4, grid-cols-5, grid-cols-6, grid-cols-7, grid-cols-8, grid-cols-9, grid-cols-10, grid-cols-11, grid-cols-12

/* Grid 行数 */
grid-rows-1, grid-rows-2, grid-rows-3, grid-rows-4, grid-rows-5, grid-rows-6

/* 列跨度 */
col-span-1, col-span-2, col-span-3, col-span-4, col-span-5, col-span-6, col-span-7, col-span-8, col-span-9, col-span-10, col-span-11, col-span-12, col-span-full

/* 行跨度 */
row-span-1, row-span-2, row-span-3, row-span-4, row-span-5, row-span-6, row-span-full
```

### ✅ Flexbox 系统

```css
/* Flex 方向 */
flex-row, flex-row-reverse, flex-col, flex-col-reverse

/* Flex 包装 */
flex-wrap, flex-wrap-reverse, flex-nowrap

/* Justify Content */
justify-start, justify-end, justify-center, justify-between, justify-around, justify-evenly

/* Align Items */
items-start, items-end, items-center, items-baseline, items-stretch

/* Align Self */
self-auto, self-start, self-end, self-center, self-stretch, self-baseline
```

## 🎭 阴影系统

### ✅ 允许的阴影类名

```css
shadow-none
shadow-sm
shadow
shadow-md
shadow-lg
shadow-xl
shadow-2xl
shadow-inner
```

### ❌ 禁止的阴影用法

```jsx
// 🚫 错误 - 不存在
<div className="shadow-xs">     // 不存在
<div className="shadow-large">  // 不存在
<div className="shadow-3xl">    // 不存在
```

## 🔄 状态变体约束

### ✅ 标准状态修饰符

```css
/* 鼠标状态 */
hover:, focus:, active:, disabled:

/* 表单状态 */
checked:, invalid:, required:, optional:

/* 位置状态 */
first:, last:, odd:, even:

/* 组合状态 */
group-hover:, group-focus:, peer-checked:
```

### ✅ 正确的状态用法

```jsx
<button className="bg-blue-500 hover:bg-blue-600 focus:ring-2 focus:ring-blue-300">
<input className="border border-gray-300 focus:border-blue-500 focus:ring-1 focus:ring-blue-500">
<div className="group">
  <div className="opacity-50 group-hover:opacity-100">
```

## 💡 最佳实践

### ✅ 推荐的组合模式

```jsx
// ✅ 卡片组件
<div className="bg-white rounded-lg shadow-md p-6 hover:shadow-lg transition-shadow">

// ✅ 按钮组件
<button className="px-4 py-2 bg-blue-500 text-white rounded-md hover:bg-blue-600 focus:outline-none focus:ring-2 focus:ring-blue-300">

// ✅ 输入框组件
<input className="w-full px-3 py-2 border border-gray-300 rounded-md focus:outline-none focus:ring-2 focus:ring-blue-500 focus:border-blue-500">

// ✅ 网格布局
<div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-6">
```

### ❌ 避免的错误模式

```jsx
// 🚫 错误 - 使用不存在的类名
<div className="gap-2.5 p-1.5 shadow-xs text-xxl">

// 🚫 错误 - 自创类名
<div className="card-shadow button-primary text-huge">

// 🚫 错误 - 混合框架语法
<div className="d-flex justify-content-center"> // 这是 Bootstrap 语法
```

## 🛠️ 验证工具

### ESLint 规则建议

使用以下工具验证 Tailwind 类名：

```json
{
  "plugins": ["@tailwindcss/eslint-plugin-tailwindcss"],
  "rules": {
    "tailwindcss/classnames-order": "warn",
    "tailwindcss/no-custom-classname": "error",
    "tailwindcss/no-contradicting-classname": "error"
  }
}
```

### VS Code 扩展

- **Tailwind CSS IntelliSense**: 提供自动补全和验证
- **Headwind**: 自动排序 Tailwind 类名

## 📚 参考资源

- [Tailwind CSS 官方文档](mdc:https:/tailwindcss.com/docs)
- [Headless UI 官方文档](mdc:https:/headlessui.com)
- [Tailwind CSS 间距系统](mdc:https:/tailwindcss.com/docs/customizing-spacing)
- [Tailwind CSS 颜色系统](mdc:https:/tailwindcss.com/docs/colors)

---

**重要提醒：始终参考最新的官方文档，避免使用过时或不存在的类名！**

---
> Source: [gaoxt/zhouwenwang](https://github.com/gaoxt/zhouwenwang) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-04 -->
