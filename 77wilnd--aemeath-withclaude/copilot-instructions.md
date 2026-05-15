## aemeath-withclaude

> Q 版像素爱弥斯桌宠，与 Claude Code 实时联动。

# Aemeath Claude Code Pet

Q 版像素爱弥斯桌宠，与 Claude Code 实时联动。

## 安装

1. 启动 Aemeath.exe（宠物显示在桌面 + 托盘图标）
2. 将 [docs/hooks.json](docs/hooks.json) 合并到 `~/.claude/settings.json`（注意替换 exe 路径）
3. 将 [docs/mcp.json](docs/mcp.json) 写入 `~/.claude/.mcp.json`
4. 重启 Claude Code

## 端口

- HTTP: 127.0.0.1:9527
- MCP: 127.0.0.1:9528

## 构建

```bash
npm install
cargo build --manifest-path src-tauri/Cargo.toml --release
```

产出在 `src-tauri/target/release/`。

## 前置要求

- [Rust](https://rustup.rs/) (stable toolchain)
- [Node.js](https://nodejs.org/) >= 18
- Windows 10+

## 目录结构

```
aemeath-claude/
├── src-tauri/
│   ├── Cargo.toml              # Rust 依赖
│   ├── tauri.conf.json         # 透明窗口 / 置顶 / 托盘配置
│   ├── icons/
│   └── src/
│       ├── main.rs             # 入口，启动 HTTP + MCP + Tauri
│       ├── state.rs            # 状态机 + 气泡文案映射 + 状态锁
│       ├── http.rs             # axum HTTP Server (:9527) + hook 端点
│       ├── mcp.rs              # MCP JSON-RPC Server (:9528)
│       └── tray.rs             # 系统托盘
├── src/
│   ├── index.html              # 宠物渲染页面
│   ├── pet.css                 # 精灵 / 气泡 / 透明窗口样式
│   ├── sprite-animator.js      # CSS spritesheet 帧动画引擎
│   ├── bubble.js               # 气泡消息队列组件
│   ├── app.js                  # 主逻辑 + 拖拽 + 轮询 + 气泡锁
│   ├── spritesheet.webp        # 精灵图集 (1536x3120)
│   └── validation.json         # 帧元数据
├── docs/
│   ├── hooks.json              # hooks 配置模板
│   └── mcp.json                # MCP 配置模板
├── .claude/settings.json       # 项目级 hooks 模板（同 docs/hooks.json）
├── CLAUDE.md
├── LICENSE
└── package.json
```

---
> Source: [77wilNd/aemeath_withclaude](https://github.com/77wilNd/aemeath_withclaude) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-12 -->
