## daidai-live2d-pet

> - pixi-live2d-display

# CLAUDE.md

## 技术栈

- Electron CommonJS
- PixiJS 6
- pixi-live2d-display
- Node.js built-in test runner

## 构建与测试

```powershell
npm install
npm test
npm start
```

## 架构概览

- `src/main/main.js`：Electron 主进程，窗口、托盘、IPC、状态轮询
- `src/main/config-store.js`：用户配置读写
- `src/main/model-registry.js`：Live2D 模型验证、导入、注册表
- `src/main/status-poller.js`：AI 状态轮询 adapter
- `src/preload.js`：安全 IPC bridge
- `src/renderer/app.js`：宠物窗和管理窗共用渲染入口

## 工作规范

- 不修改成熟参考项目：`D:\呆呆工作区\其他项目\ClaudePet`
- 旧项目只读参考，迁移功能时重写为可开源、可配置模块
- 不默认打包版权不清的角色模型
- 做完至少跑 `npm test`
- 注释用中文，保持少量且解释关键边界

## 禁止项

- 不写入旧 ClaudePet 项目
- 不硬编码老公本机私有模型路径到产品逻辑
- 不把导入模型提交进仓库，除非模型 license 已确认
- 不在 renderer 开启 `nodeIntegration`

## 压缩时保留

1. 旧项目只读、新项目独立
2. 模型导入复制到 Electron userData
3. 宠物窗和管理窗共用 `src/renderer/index.html`
4. 后续 Codex/Claude Code 接入走 adapter，不串源

---
> Source: [Rosa134/daidai-live2d-pet](https://github.com/Rosa134/daidai-live2d-pet) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-15 -->
