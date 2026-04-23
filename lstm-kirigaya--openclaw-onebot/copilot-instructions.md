## openclaw-onebot

> OpenClaw OneBot 是一个 OneBot v11 协议渠道插件，用于将 QQ（通过 Lagrange.Core/go-cqhttp/NapCat 等）接入 OpenClaw Gateway。

# OpenClaw OneBot 插件开发指南

## 项目概述

OpenClaw OneBot 是一个 OneBot v11 协议渠道插件，用于将 QQ（通过 Lagrange.Core/go-cqhttp/NapCat 等）接入 OpenClaw Gateway。

## 技术栈

- **语言**: TypeScript
- **运行时**: Node.js >= 22
- **构建工具**: TypeScript Compiler (tsc)
- **包管理器**: npm

## 核心依赖

```json
{
  "dependencies": {
    "@clack/prompts": "^1.0.0",
    "cron": "^4.4.0",
    "fuse.js": "^7.1.0",
    "highlight.js": "^11.10.0",
    "marked": "^15.0.0",
    "tsx": "^4.0.0",
    "ws": "^8.17.0"
  },
  "optionalDependencies": {
    "satori": "^0.12.0",
    "sharp": "^0.33.0"
  }
}
```

## 项目结构

```
openclaw-onebot/
├── src/                    # 源代码
│   ├── index.ts           # 插件入口
│   ├── channel.ts         # OneBot 渠道实现
│   ├── connection.ts      # WebSocket 连接管理
│   ├── og-image.ts        # Markdown 转图片（satori）
│   ├── handlers/          # 消息处理器
│   │   └── process-inbound.ts
│   └── ...
├── dist/                  # 编译输出（.js）
├── test/                  # 测试脚本
│   └── render-og-image.ts
├── skills/                # OpenClaw 技能
├── themes/                # 主题文件
├── scripts/               # 工具脚本
├── package.json
├── tsconfig.json
└── openclaw.plugin.json   # 插件配置
```

## 开发工作流

### 1. 安装依赖

```bash
npm install
```

### 2. 开发模式

修改 TypeScript 源代码后需要重新编译：

```bash
npm run build
```

### 3. 测试图片生成功能

```bash
# 测试默认主题
npx tsx test/render-og-image.ts default

# 测试 dust 主题
npx tsx test/render-og-image.ts dust

# 输出位置
# test/output-render-default.png
```

### 4. 重启 OpenClaw 服务

```bash
# 停止服务
launchctl stop ai.openclaw.gateway

# 启动服务
launchctl start ai.openclaw.gateway

# 查看状态
openclaw status
```

### 5. 查看日志

```bash
# Gateway 日志
tail -f /tmp/openclaw/openclaw-$(date +%Y-%m-%d).log

# LaunchAgent 日志
tail -f /Users/kirigaya/.openclaw/logs/gateway.log
```

## 配置说明

### OpenClaw 配置

配置文件位置: `~/.openclaw/openclaw.json`

关键配置项：

```json
{
  "channels": {
    "onebot": {
      "type": "forward-websocket",
      "host": "192.168.10.1",
      "port": 3001,
      "accessToken": "your-token",
      "enabled": true,
      "requireMention": true,        // 是否需要 @ 才响应
      "renderMarkdownToPlain": true,
      "longMessageMode": "og_image", // normal | og_image | forward
      "ogImageRenderTheme": "dust",  // 图片主题
      "longMessageThreshold": 300    // 超过此字符数转图片
    }
  },
  "plugins": {
    "load": {
      "paths": [
        "/Users/kirigaya/project/qqbot/openclaw-onebot"
      ]
    }
  }
}
```

### 长消息处理模式

| 模式 | 说明 |
|------|------|
| `normal` | 正常分段发送 |
| `og_image` | 生成图片发送（需 satori + sharp） |
| `forward` | 合并转发消息 |

## 图片生成实现

### Satori 方案

使用 `satori` + `sharp` 替代 `puppeteer-core`，优势：
- 无需浏览器内核
- 启动速度快
- Serverless 友好

### 字体要求

- Satori 不支持 `.ttc` (TrueType Collection) 格式
- 使用 `.ttf` 格式字体
- macOS 推荐: `/System/Library/Fonts/Supplemental/Arial Unicode.ttf`

### Markdown 渲染

支持语法：
- `#` 标题
- `**粗体**`
- `*斜体*`
- `` `代码` `` 行内代码
- ```` ``` ```` 代码块
- `-` / `*` 列表
- `>` 引用块

## 调试技巧

### 1. 检查插件加载

```bash
openclaw status
# 查看 Channels 部分是否显示 OneBot: ON
```

### 2. 查看插件日志

```bash
# 实时查看日志
tail -f /tmp/openclaw/openclaw-$(date +%Y-%m-%d).log | grep onebot
```

### 3. 测试消息发送

```bash
openclaw message send \
  --channel onebot \
  --target group:1046693162 \
  -m "测试消息"
```

### 4. 验证编译输出

```bash
# 确保 dist/ 目录有最新编译结果
ls -la dist/
head -20 dist/index.js
```

## 常见问题

### 1. 字体加载失败

**错误**: `Unsupported OpenType signature ttcf`

**解决**: 更换为 `.ttf` 格式字体

### 2. Satori 布局错误

**错误**: `Expected <div> to have explicit "display: flex"`

**解决**: 确保所有容器元素都有 `display: flex`

### 3. 插件未生效

**排查步骤**:
1. 检查 `npm run build` 是否成功
2. 检查 `~/.openclaw/openclaw.json` 中 plugins.load.paths 是否正确
3. 重启 OpenClaw: `launchctl stop ai.openclaw.gateway && launchctl start ai.openclaw.gateway`

## 发布流程

1. 更新版本号: `package.json` 中的 `version`
2. 编译: `npm run build`
3. 提交: `git add -A && git commit -m "..." && git push origin main`
4. 打标签: `git tag -a v1.0.x -m "Release v1.0.x" && git push origin v1.0.x`

## 相关资源

- [OneBot v11 协议文档](https://github.com/botuniverse/onebot-11)
- [Satori 文档](https://github.com/vercel/satori)
- [OpenClaw 文档](https://docs.openclaw.ai)
- [Lagrange.Core](https://github.com/LagrangeDev/Lagrange.Core)

## 开发环境

- **OS**: macOS 26.2 (arm64)
- **Node.js**: v24.14.0
- **OpenClaw**: 2026.3.x

---

最后更新: 2026-03-15

---
> Source: [LSTM-Kirigaya/openclaw-onebot](https://github.com/LSTM-Kirigaya/openclaw-onebot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-21 -->
