## tauri-integration

> Easy Window 基于 Tauri 2.x 构建，将Web前端与Rust后端相结合，提供原生桌面应用体验。

# Tauri 桌面应用集成规范

## Tauri 架构概览

Easy Window 基于 Tauri 2.x 构建，将Web前端与Rust后端相结合，提供原生桌面应用体验。

### 项目结构
```
├── src/                    # Vue3 前端代码
├── src-tauri/             # Tauri/Rust 后端代码
│   ├── src/               # Rust源码
│   ├── Cargo.toml         # Rust依赖配置
│   ├── tauri.conf.json    # Tauri配置文件
│   └── icons/             # 应用图标
├── dist/                  # 构建输出
└── target/                # Rust编译输出
```

## 前端与后端通信

### 调用Tauri API
```typescript
import { invoke } from '@tauri-apps/api/core'
import { readTextFile, writeTextFile } from '@tauri-apps/plugin-fs'

// 调用Rust后端命令
const result = await invoke('my_command', { 
  param1: 'value1',
  param2: 'value2' 
})

// 文件系统操作
const content = await readTextFile('path/to/file.txt')
await writeTextFile('path/to/file.txt', 'new content')
```

### Window管理
```typescript
import { getCurrentWindow } from '@tauri-apps/api/window'

const appWindow = getCurrentWindow()

// 窗口操作
await appWindow.setTitle('新标题')
await appWindow.minimize()
await appWindow.maximize()
await appWindow.close()

// 监听窗口事件
appWindow.listen('tauri://close-requested', () => {
  console.log('用户尝试关闭窗口')
})
```

## 文件系统操作

### 文件读写
```typescript
import { 
  readTextFile, 
  writeTextFile, 
  exists,
  createDir,
  copyFile
} from '@tauri-apps/plugin-fs'
import { join, appDataDir } from '@tauri-apps/api/path'

// 获取应用数据目录
const appDataPath = await appDataDir()
const configPath = await join(appDataPath, 'config.json')

// 检查文件是否存在
if (await exists(configPath)) {
  const config = await readTextFile(configPath)
  console.log('配置文件内容:', config)
}

// 写入配置文件
await writeTextFile(configPath, JSON.stringify(configData))
```

### 对话框操作
```typescript
import { open, save } from '@tauri-apps/plugin-dialog'

// 打开文件选择对话框
const selected = await open({
  multiple: false,
  filters: [{
    name: 'Vue文件',
    extensions: ['vue']
  }]
})

// 保存文件对话框
const filePath = await save({
  filters: [{
    name: '项目文件',
    extensions: ['json']
  }]
})
```

## 应用配置管理

### tauri.conf.json 关键配置
```json
{
  "app": {
    "name": "Easy Window",
    "version": "1.0.0"
  },
  "build": {
    "frontendDist": "../dist"
  },
  "bundle": {
    "identifier": "com.easywindow.app",
    "windows": {
      "certificateThumbprint": null,
      "digestAlgorithm": "sha256",
      "timestampUrl": ""
    }
  },
  "plugins": {
    "fs": {
      "scope": [
        "$APPDATA/**",
        "$DOCUMENT/**"
      ]
    }
  }
}
```

### 安全策略
Tauri采用严格的安全策略，需要明确配置允许的API和文件访问范围：

```json
{
  "permissions": [
    "fs:default",
    "dialog:default",
    "window:default"
  ]
}
```

## 自动更新系统

### 更新检查
```typescript
import { check } from '@tauri-apps/plugin-updater'

async function checkForUpdates() {
  try {
    const update = await check()
    if (update?.available) {
      console.log('发现新版本:', update.version)
      console.log('更新日志:', update.body)
      
      // 下载并安装更新
      await update.downloadAndInstall()
      
      // 重启应用
      await relaunch()
    }
  } catch (error) {
    console.error('检查更新失败:', error)
  }
}
```

### 更新配置
在 `tauri.conf.json` 中配置更新服务器：

```json
{
  "updater": {
    "active": true,
    "endpoints": [
      "https://api.yourdomain.com/updates/{{target}}/{{current_version}}"
    ],
    "dialog": true,
    "pubkey": "YOUR_PUBLIC_KEY"
  }
}
```

## 打包和分发

### 开发模式运行
```bash
# 启动开发服务器
npm run tauri:dev

# 仅构建前端
npm run dev
```

### 生产构建
```bash
# 构建应用
npm run tauri:build

# 构建特定平台
npm run tauri build -- --target x86_64-pc-windows-msvc
```

### 构建配置优化
```json
{
  "bundle": {
    "targets": ["msi", "nsis"],
    "resources": ["assets/*"],
    "externalBin": [],
    "copyright": "© 2024 Easy Window Team",
    "category": "DeveloperTool",
    "shortDescription": "桌面UI生成工具",
    "longDescription": "一款基于Vue3+Tauri的可视化桌面UI设计工具"
  }
}
```

## 开发最佳实践

### 1. 错误处理
```typescript
try {
  const result = await invoke('risky_command')
  return result
} catch (error) {
  console.error('Tauri命令执行失败:', error)
  // 显示用户友好的错误消息
  ElMessage.error('操作失败，请重试')
  throw error
}
```

### 2. 跨平台兼容性
```typescript
import { platform } from '@tauri-apps/plugin-os'

const currentPlatform = await platform()
if (currentPlatform === 'windows') {
  // Windows特定逻辑
} else if (currentPlatform === 'macos') {
  // macOS特定逻辑
}
```

### 3. 性能优化
- 避免频繁的前后端通信
- 缓存常用的Tauri API调用结果
- 使用批量操作减少IPC开销

### 4. 调试技巧
```typescript
// 开发模式下启用控制台
if (import.meta.env.DEV) {
  document.addEventListener('contextmenu', (e) => {
    e.preventDefault()
  })
}

// 监听Tauri事件进行调试
import { listen } from '@tauri-apps/api/event'

listen('tauri://window-created', (event) => {
  console.log('窗口创建事件:', event)
})
```

## 原生功能集成

### 系统托盘
```typescript
import { TrayIcon } from '@tauri-apps/api/tray'

const tray = await TrayIcon.new({
  icon: 'icons/icon.png',
  tooltip: 'Easy Window'
})

tray.on('click', () => {
  // 点击托盘图标的处理逻辑
})
```

### 全局快捷键
```typescript
import { register } from '@tauri-apps/plugin-global-shortcut'

await register('CommandOrControl+Shift+E', () => {
  console.log('快捷键被触发')
})
```

### 系统通知
```typescript
import { 
  isPermissionGranted, 
  requestPermission, 
  sendNotification 
} from '@tauri-apps/plugin-notification'

let permissionGranted = await isPermissionGranted()
if (!permissionGranted) {
  const permission = await requestPermission()
  permissionGranted = permission === 'granted'
}

if (permissionGranted) {
  sendNotification({
    title: 'Easy Window',
    body: '操作完成！'
  })
}
```

---
> Source: [suyancc/easy_window](https://github.com/suyancc/easy_window) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-07 -->
