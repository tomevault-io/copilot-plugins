## reacttest

> - Tailwind CSS className을 작성할 때 일관된 순서를 유지한다.

# Tailwind CSS 클래스 순서 규칙

<prompt>
<system>
- Tailwind CSS className을 작성할 때 일관된 순서를 유지한다.
- 코드의 가독성과 유지보수성을 위해 정해진 순서를 엄격히 따른다.
- 클래스 순서: Layout → Spacing → Color 순으로 작성한다.
</system>

<rule>
## 클래스 작성 순서

### 1. Layout (레이아웃)

- Display: `block`, `inline`, `flex`, `grid`, `hidden` 등
- Position: `static`, `relative`, `absolute`, `fixed`, `sticky` 등
- Top/Right/Bottom/Left: `top-0`, `right-4`, `bottom-2`, `left-auto` 등
- Float: `float-left`, `float-right`, `float-none` 등
- Clear: `clear-left`, `clear-right`, `clear-both` 등
- Z-index: `z-0`, `z-10`, `z-50` 등

### 2. Flexbox & Grid

- Flex Direction: `flex-row`, `flex-col`, `flex-row-reverse` 등
- Flex Wrap: `flex-wrap`, `flex-nowrap`, `flex-wrap-reverse` 등
- Flex: `flex-1`, `flex-auto`, `flex-initial`, `flex-none` 등
- Flex Grow/Shrink: `flex-grow`, `flex-shrink` 등
- Order: `order-1`, `order-2` 등
- Grid Template: `grid-cols-12`, `grid-rows-6` 등
- Grid Column/Row: `col-span-2`, `row-span-3` 등
- Gap: `gap-4`, `gap-x-2`, `gap-y-6` 등
- Justify/Align: `justify-center`, `items-center`, `content-between` 등

### 3. Box Model

- Width: `w-full`, `w-1/2`, `w-96` 등
- Height: `h-screen`, `h-64`, `h-auto` 등
- Min/Max Width/Height: `min-w-0`, `max-h-screen` 등

### 4. Spacing (간격)

- Margin: `m-4`, `mx-auto`, `mt-2`, `mb-6` 등
- Padding: `p-4`, `px-6`, `py-2`, `pt-8` 등

### 5. Typography

- Font Family: `font-sans`, `font-serif`, `font-mono` 등
- Font Size: `text-xs`, `text-sm`, `text-base`, `text-lg` 등
- Font Weight: `font-thin`, `font-normal`, `font-bold` 등
- Text Align: `text-left`, `text-center`, `text-right` 등
- Text Color: `text-gray-900`, `text-blue-500` 등
- Text Decoration: `underline`, `line-through`, `no-underline` 등
- Line Height: `leading-tight`, `leading-normal`, `leading-loose` 등

### 6. Color (색상)

- Background Color: `bg-white`, `bg-gray-100`, `bg-blue-500` 등
- Background Image: `bg-gradient-to-r`, `bg-cover`, `bg-center` 등
- Border Color: `border-gray-300`, `border-red-500` 등

### 7. Border

- Border Width: `border`, `border-2`, `border-t`, `border-r` 등
- Border Radius: `rounded`, `rounded-lg`, `rounded-full` 등
- Border Style: `border-solid`, `border-dashed`, `border-dotted` 등

### 8. Effects

- Box Shadow: `shadow`, `shadow-md`, `shadow-lg`, `shadow-none` 등
- Opacity: `opacity-0`, `opacity-50`, `opacity-100` 등
- Mix Blend Mode: `mix-blend-multiply`, `mix-blend-screen` 등

### 9. Transforms

- Transform: `transform`, `transform-gpu`, `transform-none` 등
- Scale: `scale-50`, `scale-100`, `scale-110` 등
- Rotate: `rotate-45`, `rotate-90`, `rotate-180` 등
- Translate: `translate-x-4`, `translate-y-2` 등
- Skew: `skew-x-12`, `skew-y-6` 등

### 10. Transitions & Animation

- Transition: `transition`, `transition-all`, `transition-colors` 등
- Duration: `duration-75`, `duration-300`, `duration-1000` 등
- Ease: `ease-in`, `ease-out`, `ease-in-out` 등
- Animation: `animate-spin`, `animate-ping`, `animate-bounce` 등

### 11. Interactivity

- Cursor: `cursor-pointer`, `cursor-not-allowed` 등
- User Select: `select-none`, `select-text`, `select-all` 등
- Pointer Events: `pointer-events-none`, `pointer-events-auto` 등
- Resize: `resize`, `resize-none`, `resize-x`, `resize-y` 등

### 12. Responsive & State Modifiers (마지막)

- Responsive: `sm:`, `md:`, `lg:`, `xl:`, `2xl:` 등
- State: `hover:`, `focus:`, `active:`, `disabled:` 등
- Dark Mode: `dark:` 등
  </rule>

<example>
❌ 잘못된 예시:
```jsx
<div className="text-white bg-blue-500 p-4 flex rounded-lg">
```

✅ 올바른 예시:

```jsx
<div className="flex p-4 bg-blue-500 text-white rounded-lg">
```

❌ 잘못된 예시:

```jsx
<button className="hover:bg-blue-600 text-white bg-blue-500 px-4 py-2 rounded font-medium">
```

✅ 올바른 예시:

```jsx
<button className="px-4 py-2 bg-blue-500 text-white font-medium rounded hover:bg-blue-600">
```

❌ 잘못된 예시:

```jsx
<div className="shadow-lg bg-white border mt-4 p-6 rounded-xl border-gray-200">
```

✅ 올바른 예시:

```jsx
<div className="mt-4 p-6 bg-white border border-gray-200 rounded-xl shadow-lg">
```

</example>

<format>
클래스 작성 시 다음 순서를 엄격히 준수:
1. Layout (display, position, float, z-index)
2. Flexbox & Grid (flex, grid, gap, justify, align)
3. Box Model (width, height, min/max)
4. Spacing (margin, padding)
5. Typography (font, text)
6. Color (background, text color)
7. Border (width, radius, style, color)
8. Effects (shadow, opacity)
9. Transforms (scale, rotate, translate)
10. Transitions & Animation
11. Interactivity (cursor, select)
12. Responsive & State Modifiers (sm:, hover:, etc.)
</format>
</prompt>
description:
globs:
alwaysApply: false
---

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/umingming)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/umingming)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
