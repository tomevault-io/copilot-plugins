## metal-slug-game

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

合金弹头 (Metal Slug) 横版射击小游戏，纯前端 HTML5 Canvas + JavaScript 实现，无需构建工具或外部依赖。

## Commands

```bash
# 本地运行：直接在浏览器打开 index.html
start index.html           # Windows
open index.html            # macOS

# 部署：推送到 GitHub 后自动通过 GitHub Actions 部署到 Pages
git push
```

## Architecture

单文件游戏架构，所有逻辑集中在 `game.js` (~43KB)，采用模块化函数组织：

### Game Loop
- `gameLoop()` — `requestAnimationFrame` 主循环，调用 `update()` + `draw()`
- `update()` — 状态机处理 (menu/playing/gameover/victory)
- `draw()` — 渲染层，按深度顺序绘制

### Entity Systems
- **Player**: `player` 对象 + `updatePlayer()`, `drawPlayer()`, `damagePlayer()`, `resetPlayer()`
- **Enemies**: `enemies[]` 数组 + `createEnemy(type,x,y)`, `updateEnemies()`, `drawEnemies()`, `damageEnemy()`
- **Boss**: `boss` 对象 + `createBoss()`, `updateBoss()`, `drawBoss()`, `damageBoss()`
- **Bullets**: `bullets[]` + `createBullet()`, `updateBullets()`, `drawBullets()`
- **Grenades**: `grenades[]` + `throwGrenade()`, `updateGrenades()`
- **Particles**: `particles[]` + `spawnParticles()`, `updateParticles()`, `drawParticles()`

### Level Data
- `platforms[]` — 地形碰撞数据 (地面 + 平台)
- `obstacles[]` — 装饰性障碍物
- `spawnLevelEnemies()` — 关卡敌人生成脚本

### Core Systems
- **Input**: `keys{}` 键盘状态 + 触摸按钮 `data-key` 映射
- **Camera**: `cameraX` 横向卷轴，`updateCamera()` 玩家跟随
- **Sound**: Web Audio API 合成音效，`playSound(type)`
- **Collision**: AABB 矩形碰撞检测，分散在各 entity update 函数中

### Constants
- `W=960, H=540` — Canvas 尺寸
- `GROUND_Y` — 地面 Y 坐标
- `GRAVITY=0.6` — 重力加速度
- `LEVEL_W=5200` — 关卡总宽度

## Controls
- 键盘: A/D 移动, W 跳跃, S 蹲下, J/空格 射击, K 手雷
- 触摸: D-pad 方向按钮 + 射击/手雷 按钮 (移动端自动显示)

## Deployment

GitHub Actions 自动部署 (`/.github/workflows/deploy.yml`)：
- push 到 main 分支触发
- 部署到 `https://<username>.github.io/<repo-name>/`

需在仓库 Settings > Pages > Source 选择 "GitHub Actions"。

---
> Source: [SongZeHNU/metal-slug-game](https://github.com/SongZeHNU/metal-slug-game) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-02 -->
