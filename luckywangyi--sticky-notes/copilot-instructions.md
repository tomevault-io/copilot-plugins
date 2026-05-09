## ui-design

> 便签应用 UI 设计规范 — 所有界面修改必须遵守


# 便签应用 UI 设计规范

## 核心原则
- 彻底禁止 AI 默认蓝白风格
- 所有 React 组件文件(.tsx)必须 `import React from 'react'`

## 窗口质感
- macOS: Electron `transparent: true` + `vibrancy: 'under-window'`
- Windows: 实色背景 `#e8e4df` + CSS `backdrop-filter: blur(20px) saturate(180%)` 模拟磨砂
- 边框: `0.5px solid rgba(255, 255, 255, 0.18)`
- 圆角: `border-radius: 10px`

## 配色 — 莫兰迪色系（低饱和度半透明）
五种便签底色，不可更改：
| 名称     | 背景色 (半透明)                  | 实色         |
|----------|----------------------------------|--------------|
| 雾灰玫瑰 | `rgba(198, 178, 175, 0.75)`     | `#c6b2af`    |
| 烟青绿   | `rgba(172, 192, 183, 0.75)`     | `#acc0b7`    |
| 暮光蓝灰 | `rgba(175, 183, 198, 0.75)`     | `#afb7c6`    |
| 亚麻米   | `rgba(205, 197, 182, 0.75)`     | `#cdc5b6`    |
| 淡藕粉   | `rgba(208, 188, 192, 0.75)`     | `#d0bcc0`    |

严禁使用明亮色、纯白、纯蓝、明黄等高饱和色。

## 交互
- 顶部工具栏默认隐藏: `opacity: 0; transform: translateY(-4px)`
- hover 时过渡显示: `transition: opacity 0.25s ease, transform 0.25s ease`
- 图标库: Lucide React，统一 `strokeWidth={1}` (Thin 风格)

## 字体
```css
font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", "PingFang SC",
             "Microsoft YaHei", "Noto Sans SC", sans-serif;
letter-spacing: -0.02em;
```
强制系统无衬线字体，禁止引入网络字体。

## 窗口尺寸（参考便签应用）
- 默认: 380×540
- 最小: 300×360
- 无边框窗口 (`frame: false`)
- 关闭按钮最小化到系统托盘，不退出应用

## 列表项设计（禁止 AI 化设计）
- 禁止使用左侧/右侧彩色竖条 (border-left/border-right colored bar)
- 便签列表项使用对应莫兰迪色的低透明度版本 (alpha ~0.18) 作为整卡片背景
- 参考 Google Keep 的淡色卡片风格
- 圆角统一 8px，卡片间距 2px

## UI 修改检查清单（每次修改必须逐项确认）
1. **重叠检查**: 新增/移动的元素是否与同区域已有元素（文字、图标、按钮）在任何状态（默认、hover、active、focus）下重叠？
2. **可点击区域**: 按钮/链接的点击热区是否 ≥ 24×24px？是否被其他元素遮挡？
3. **状态切换**: hover 显示的元素是否会挡住原有内容？若是，原有内容应隐藏或位移让出空间。
4. **间距一致**: 元素间距是否与同级元素保持一致？参考既有间距值（2px/4px/6px/8px/10px/14px）。
5. **层级合理**: `z-index` 是否正确？弹出菜单 > 浮动按钮 > 内容层。
6. **响应式**: 窗口缩到最小尺寸（300×360）时布局是否正常？文字是否溢出？
7. **过渡动画**: 显示/隐藏是否有 `transition`？禁止突变闪烁。

## 禁止事项
- 禁止将颜色改为不透明实色（保留 rgba 半透明 + backdrop-filter 磨砂效果）
- 禁止删除 `backdrop-filter` 属性
- 禁止增大窗口默认尺寸超过 500×700
- 禁止添加 AI 默认的蓝色主题色
- 禁止移除莫兰迪色板中的任何颜色

---
> Source: [luckywangyi/sticky-notes](https://github.com/luckywangyi/sticky-notes) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-04 -->
