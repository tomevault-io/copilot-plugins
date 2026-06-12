## joker

> Balatro 风格网页卡牌游戏，第 1 轮核心循环。Vue 3 (Composition API + `<script setup>`) + Vite 6 + GSAP。

# CLAUDE.md

## 项目定位

Balatro 风格网页卡牌游戏，第 1 轮核心循环。Vue 3 (Composition API + `<script setup>`) + Vite 6 + GSAP。

## 常用命令

```bash
# 启动开发服务器（需用 Cursor 内置 node，系统无全局 node）
/Applications/Cursor.app/Contents/Resources/app/resources/helpers/node node_modules/.bin/vite

# 构建
/Applications/Cursor.app/Contents/Resources/app/resources/helpers/node node_modules/.bin/vite build

# 在浏览器中打开
open http://localhost:5173
```

## 目录约定

- `src/gameLogic.js` — 纯函数，零副作用，所有牌型/得分逻辑在此
- `src/App.vue` — 状态机入口，状态：`playing` / `shop` / `won` / `lost`
- `src/components/` — UI 组件，每个组件只负责显示，事件向上 emit
- `src/assets/main.css` — CSS 变量定义（`--bg-deep`、`--gold` 等），全局样式

## 设计约束

- 主题：深蓝水彩（`#0a1438` 底色），禁止绿色毡布、紫色背景
- 字体：Press Start 2P（标题）、VT323（数字）、Inter（中文）
- 动画通过 GSAP 控制，速度倍率由 `settings.animSpeed` 决定
- 设置持久化到 `localStorage`，key 为 `balatro.settings`

## Node 运行时说明

系统无全局 `node`，使用 Cursor 内置 Node v22：
`/Applications/Cursor.app/Contents/Resources/app/resources/helpers/node`

---
> Source: [cj920512-prog/joker](https://github.com/cj920512-prog/joker) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-12 -->
