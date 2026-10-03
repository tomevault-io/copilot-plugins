## joker-game

> 小丑牌是一个 Balatro 风格的网页扑克策略游戏，Vue 3 + Vite + GSAP 实现，无后端依赖。

# CLAUDE.md

## 项目定位

小丑牌是一个 Balatro 风格的网页扑克策略游戏，Vue 3 + Vite + GSAP 实现，无后端依赖。

## 目录结构

```
joker-game/
  src/
    gameLogic.js        # 牌型识别、Joker 效果、AI 枚举（纯函数，无副作用）
    App.vue             # 根组件，持有全部游戏状态和动画编排
    components/
      PlayingCard.vue   # 单张扑克牌渲染
      HandArea.vue      # 手牌区 + 操作按钮
      PlayArea.vue      # 出牌区（含绝对定位牌堆）
      JokerRow.vue      # Joker 槽位（5 个）
      SideBar.vue       # 左侧信息面板
      ShopView.vue      # 商店覆盖层
      EndView.vue       # 通关/失败结算
      SettingsModal.vue # 设置弹窗，持久化到 localStorage['balatro.settings']
  public/               # 静态资源
  vite.config.js        # base 路径：本地 './' / GitHub Pages '/joker-game/'
01-第1轮-PRD-小丑牌核心循环.html   # 需求文档
01-第1轮-DESIGN-小丑牌核心循环.html # 设计规范
```

## 常用命令

```bash
cd joker-game
npm install          # 安装依赖
npm run dev          # 开发服务器
npm run build        # 生产构建
DEPLOY_TARGET=pages npm run build  # GitHub Pages 构建
```

## 关键约定

- 字体：Press Start 2P 仅用于英文装饰，VT323 仅用于数字显示，中文全用 Inter+PingFang SC
- 颜色：深蓝主题 #0a1438 → #1a2858，不用紫色或绿毡
- 动画时长受 `animSpeedMult`（慢1.5×/普通1×/快0.6×）控制
- gameLogic.js 保持纯函数，所有副作用（DOM、动画）在 App.vue 内处理

---
> Source: [ankhzw1876/joker-game](https://github.com/ankhzw1876/joker-game) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-03 -->
