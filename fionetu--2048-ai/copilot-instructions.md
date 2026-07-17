## 2048-ai

> - **项目位置**：`D:\projectwork\2048-ai`

# 2048 AI Solver — 项目上下文

## 当前状态

- **项目位置**：`D:\projectwork\2048-ai`
- **技术栈**：React 19 + TypeScript + Vite + Tailwind CSS v3
- **当前阶段**：核心开发与本地验证已完成，等待 GitHub 仓库创建和 Pages 部署
- **目标**：Bindo Labs 工程师面试题交付物

## 已完成

- [x] Vite + React + TypeScript + Tailwind CSS 项目初始化
- [x] 2048 核心游戏逻辑（移动、合并、生成新块、胜负判定）
- [x] 桌面键盘（方向键 / WASD）与移动端触屏滑动控制
- [x] 分数与最高分 LocalStorage 持久化
- [x] AI 引擎：Expectimax + 启发式评估（蛇形权重、单调性、平滑度、最大块角落奖励）
- [x] AI 自动运行模式（慢 / 中 / 快三档）
- [x] Hint 提示箭头
- [x] 响应式 UI + 方块移动 / 合并 / 出现动画
- [x] GitHub Actions 自动部署到 GitHub Pages
- [x] `npm run build` 构建成功
- [x] Git 仓库初始化，`main` 分支已有提交

## AI 表现

- Expectimax 深度 3
- 本地批量测试：5 局 4 胜到达 2048
- 自动模式可稳定演示 AI 合成 2048 的过程

## 常用命令

```bash
# 本地开发
npm run dev

# 生产构建
npm run build

# 预览生产构建
npm run preview

# 推送到 GitHub（需先创建仓库并添加 remote）
git push -u origin main
```

## GitHub Pages 部署步骤

1. 在 GitHub 创建仓库 `2048-ai`
2. 添加 remote 并推送 `main` 分支
3. 仓库 Settings → Pages → Source 选择 **GitHub Actions**
4. 访问 `https://你的用户名.github.io/2048-ai/`

## 关键文件

- `src/utils/board.ts` — 棋盘移动、合并、生成新块、胜负判定
- `src/ai/expectimax.ts` — Expectimax 搜索算法
- `src/ai/heuristics.ts` — 启发式评估函数
- `src/hooks/useGameLogic.ts` — 核心游戏状态管理
- `src/hooks/useAI.ts` — AI 自动运行与 Hint
- `src/components/Board.tsx` + `Tile.tsx` — 棋盘与动画方块
- `.github/workflows/deploy.yml` — GitHub Pages CI/CD

## 下一步（待办）

1. 创建 GitHub 仓库并推送代码
2. 开启 GitHub Pages
3. 录制项目演示视频
4. 整理 AI 对话历史作为交付物

## 注意事项

- `vite.config.ts` 中已配置 `base: '/2048-ai/'`，必须与 GitHub 仓库名一致
- 本地构建产物在 `dist/` 目录，已被 `.gitignore` 忽略
- 项目是纯前端，无后端依赖

---
> Source: [fionetu/2048-ai](https://github.com/fionetu/2048-ai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-17 -->
