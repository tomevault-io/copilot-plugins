## secs-simulator

> **Generated:** 2026-01-12

# SECS Simulator 开发指南

**Generated:** 2026-01-12
**Stack:** Electron + Vue 3 + TypeScript + Arco Design + secs4js

## 项目架构

```
secs-simulator/
├── frontend/              # Vue 3 前端
│   └── src/
│       ├── api/           # IPC 通信定义
│       ├── components/    # 共享组件
│       ├── router/        # 路由
│       ├── utils/         # 工具
│       └── views/home/    # 主页面
│           ├── components/   # 页面组件 (11 files)
│           ├── composables/  # 业务逻辑 (6 files)
│           └── types.ts      # 类型定义
├── electron/              # Electron 后端
│   ├── controller/        # IPC 处理器 (5 files)
│   ├── service/           # 业务逻辑 (8 files)
│   ├── config/            # 配置
│   └── preload/           # 预加载脚本
├── public/electron/       # 打包后文件
└── sml/                   # SML 协议文件
```

## 核心模块

| 模块 | 路径 | 职责 |
|------|------|------|
| Engine | `electron/service/engine.js` + `frontend/composables/useEngine.ts` | SECS/GEM 引擎管理 |
| SML File | `electron/service/smlFile.js` | SML 文件解析 |
| AutoReply | `electron/service/autoReply.js` | 自动回复逻辑 |
| AutoFlow | `electron/service/autoflow.js` | 自动化流程执行 |
| EventBind | `electron/service/eventBind.js` | 事件绑定生成 |

## 开发命令

```bash
# 开发 (同时启动前后端)
npm run dev

# 前端独立开发
npm run dev-frontend

# 构建
npm run build-frontend && npm run build-electron

# 类型检查
cd frontend && npm run type-check
```

## 代码约定

### TypeScript 严格模式
- `strict: true`
- `noUnusedLocals: true`
- `noUnusedParameters: true`
- 路径别名: `@/*` → `./src/*`

### 命名规范
- **Composables**: `useXxx()` (camelCase)
- **组件**: `XxxModal.vue` (PascalCase)
- **类型**: `XxxData`, `XxxItem` (PascalCase)
- **事件处理**: `handleXxx()`, `onXxx()`

### IPC 通信
```typescript
// 前端调用
import { ipc } from '@/utils/ipcRenderer'
import { ipcApiRoute } from '@/api'
await ipc.invoke(ipcApiRoute.engineStart, { config })

// 后端定义
ipcMain.handle(ipcApiRoute.engineStart, async (event, args) => {
  return await engineService.start(args, event)
})
```

### 日志规范
```javascript
logger.info('✅ [method] 操作成功')
logger.error('❌ [method] 错误信息:', error)
logger.debug('📌 [method] 调试信息')
```

## SECS/GEM 协议

### 消息处理优先级
1. **AutoReply 脚本** - 自定义 JavaScript 逻辑
2. **SML 文件匹配** - `S{stream}F{func+1}` 文件
3. **默认回复** - 奇数函数的 `L()` 响应

### 自动回复脚本
```javascript
async function handler(comingMsg, filePaths) {
  // comingMsg: { stream, func, wBit, body }
  // filePaths: SML 文件路径列表
  return targetFilePath  // 返回文件路径用于回复
}
```

## 质量保证

### 提交前检查
- [ ] `npm run type-check` 通过
- [ ] 应用正常启动
- [ ] IPC 通信正常
- [ ] 无未使用导入/变量

### 禁止模式
- ❌ `as any`, `@ts-ignore` 类型断言
- ❌ 空 catch 块 `catch(e) {}`
- ❌ 直接修改 `engine.config` 而不通过服务

## 子目录 AGENTS.md

- `frontend/src/views/home/composables/` - Composables 详细规范
- `frontend/src/views/home/components/` - 组件开发指南
- `electron/service/` - 服务层开发规范

---
> Source: [LostCat-Qian/secs-simulator](https://github.com/LostCat-Qian/secs-simulator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-03 -->
