## cc-web-manager-open

> 公网 Web 管理面板，控制 GPU 服务器上的 Claude Code。

# Claude Code Web Manager

公网 Web 管理面板，控制 GPU 服务器上的 Claude Code。

## 快速启动

```bash
CC_WEB_PASSWORD=yourpassword ./start.sh
```

## 环境变量

| 变量 | 默认值 | 说明 |
|------|--------|------|
| CC_WEB_PASSWORD | (必需) | 登录密码 |
| CC_SSH_HOST | localhost | GPU 服务器 SSH 地址 |
| CC_SSH_PORT | 2222 | GPU 服务器 SSH 端口 |
| CC_SSH_USER | root | SSH 用户名 |
| CC_MAX_PARALLEL | 3 | 最大并行任务数 |
| CC_CLAUDE_PATH | /root/.local/bin/claude | GPU 上 Claude Code 路径 |
| CC_WEB_PORT | 8080 | Web 服务端口 |

## 注意事项

- 修改 `static/app.js` 或 `static/style.css` 后，必须同步更新 `static/index.html` 中的 `?v=` 版本号，否则浏览器会使用缓存的旧文件
- `claude -p --output-format stream-json` 在 v2.1.44+ 必须搭配 `--verbose`
- interactive 模式进程退出应标记 `idle` 而非 `failed`

## Release Note 更新流程

每次功能更新时，需要同步更新 `static/app.js` 顶部的版本信息：

1. 修改 `APP_VERSION` 变量为新版本号（语义化版本 vX.Y.Z）
2. 在 `RELEASE_NOTES` 数组**开头**插入新版本条目，包含 `version`、`date`、`changes` 数组
3. 同步更新 `static/index.html` 中的 `?v=` 缓存版本号

## 架构

手机/浏览器 → FastAPI (8080) → SSH → GPU 服务器 (tmux + claude)

---
> Source: [liaoyiqiao/cc-web-manager-open](https://github.com/liaoyiqiao/cc-web-manager-open) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-25 -->
