## termcat

> > 架构详情、目录结构、模块关系、消息协议、数据流请参考 [claude_refs/ARCHITECTURE.md](claude_refs/ARCHITECTURE.md)。

# TermCat Client - Claude Code Guide

> 架构详情、目录结构、模块关系、消息协议、数据流请参考 [claude_refs/ARCHITECTURE.md](claude_refs/ARCHITECTURE.md)。
> 本文件仅包含开发规范要点，避免重复。

---

## 开发规范

### IPC 通信

```typescript
// Main 进程注册
ipcMain.handle('channel', async (event, ...args) => { ... });

// Preload 暴露
contextBridge.exposeInMainWorld('electronAPI', {
  module: { method: (...args) => ipcRenderer.invoke('channel', ...args) }
});

// Renderer 调用
const result = await window.electronAPI.module.method(...args);
```

### 类型安全

- 所有函数必须有类型注解
- 避免 `any`，优先使用 `unknown`
- 数据结构使用 interface 定义

### 错误处理

- 所有 async 操作必须 try-catch
- 提供用户可读的错误消息
- 详细错误信息记录到日志

### 注释语言

- 所有代码注释必须使用英文（行内、块注释、JSDoc、TODO）
- 文档文件（.md）不受限制

### 日志规范

```typescript
const log = logger.withFields({ module: LOG_MODULE.SSH });
log.info('ssh.connection.established', 'SSH connected', { session_id, host });
```

**必须记录：** 用户操作（按钮点击、导航、表单提交）、连接事件、WebSocket 消息

**禁止记录：** 按键事件、鼠标移动、密码/Token/私钥

### 国际化

- 禁止硬编码中文/英文文本
- 宿主使用 `useI18n()` → `t.模块.键名`
- 插件独立管理翻译（`locales/` + `useT()`）
- 暂时只需实现中文版本

### 性能规则

- 弹窗/非首屏组件必须 `React.lazy` 懒加载
- 高频事件（>10次/秒）禁止直接 `setState`，使用 rAF 或 debounce
- localStorage 写入必须防抖
- 新依赖（>30KB）需在 `vite.config.ts` 的 `manualChunks` 中分包
- 宿主代码禁止直接 import 插件模块（使用 `events.ts` 事件常量）

### 层级架构

```
base (http/logger/i18n)
  ↓
core (license/commerce/auth/ai-agent/terminal)
  ↓
features (auth/extensions/settings/terminal)
  ↓
plugins/builtin (ai-ops/file-browser/...)
  ↓
plugins/external (local-ops-aiagent)
```

- 依赖只能向下，禁止向上引用
- feature 层不得硬编码插件业务逻辑
- 插件通过 `pluginData` 声明元数据，feature 层通用渲染

---

## 常用操作

```bash
npm run dev        # 开发（Vite + Electron HMR）
npm run build      # 构建
npm run build:mac  # macOS 打包
```

### 新增 API 调用

在 `src/base/http/api.ts` 的 ApiService 类中添加方法，使用 `this.rpc()` 模式。

### 新增 IPC 通道

1. Main 进程 → `src/main/main.ts` 注册 `ipcMain.handle()`
2. Preload → `src/preload/preload.ts` 暴露到 `window.electronAPI`
3. Renderer → 直接调用

### 新增内置插件

1. `src/plugins/builtin/` 下创建目录
2. 在 `src/plugins/builtin/index.ts` 注册
3. 事件常量定义在 `src/plugins/builtin/events.ts`

---

## 参考文档

| 文档 | 内容 |
|------|------|
| [claude_refs/ARCHITECTURE.md](claude_refs/ARCHITECTURE.md) | 完整架构、目录结构、模块关系、消息协议、数据流 |
| `.claude/i18n-rules.md` | 多语言开发详细规范 |

---
> Source: [termcat-ai/termcat](https://github.com/termcat-ai/termcat) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-16 -->
