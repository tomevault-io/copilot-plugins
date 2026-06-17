## mcp-usage

> 如何使用 MCP Context Server 获取项目上下文，在 Cursor 中编写代码时必须先查询项目状态


# MCP Context Server 使用指南

## 什么是 MCP Context Server

这是一个给 Cursor AI 用的"项目说明书 API"。它让 AI 在写代码之前先了解项目实际状态，避免 API 路径猜错、组件名写错等常见幻觉问题。

## 连接状态

MCP Server 已配置在 `.cursor/mcp.json`，Cursor 启动时自动连接。

## 如何使用

### 在 Chat 中查询项目状态

在 Cursor Chat 对话框中，使用 `@second-brain-context` 召唤 MCP 工具：

1. 打开 Cursor Chat (Cmd+L)
2. 输入 `@second-brain-context` 或直接输入工具名
3. 选择需要的工具获取上下文

### 可用的工具

| 工具名 | 返回内容 |
|--------|----------|
| `list_components` | 前端所有 Vue 组件及 props/emits 定义 |
| `list_views` | 前端所有页面视图 |
| `list_api_files` | 前端 API 封装文件及导出函数 |
| `list_composables` | 前端 composables/hooks |
| `list_backend_routes` | 后端所有 API 路由和端点 |
| `get_data_model` | Note、Tag 等数据接口定义 |
| `get_tag_colors` | 标签预设颜色池 |
| `get_project_summary` | 完整项目摘要（推荐先用这个） |

## 推荐工作流

### 写新功能之前

**不要**直接让 AI 写代码。先获取上下文：

```
在 Cursor Chat 中：
@second-brain-context 调用 get_project_summary
→ AI 了解了项目实际状态
→ 再让 AI 生成代码
```

### 示例：添加标签筛选功能

**错误做法**（容易出错）：
```
帮我加一个标签筛选功能
→ AI 猜测 API 路径，可能写错
```

**正确做法**：
```
@second-brain-context 调用 get_project_summary
@second-brain-context 调用 list_backend_routes
→ 看到 GET /api/notes 支持 tag 参数
→ 再让 AI 生成代码
```

## MCP Server 调试

```bash
# 本地测试 MCP Server
node scripts/mcp-context-server.js

# HTTP 模式（curl 测试）
MCP_HTTP=true node scripts/mcp-context-server.js
curl -X POST http://localhost:3456 -d '{"jsonrpc":"2.0","id":1,"method":"tools/call","params":{"name":"get_project_summary","arguments":{}}}'
```

## 常见问题

Q: Cursor 连接 MCP 失败？
A: 确认 `.cursor/mcp.json` 在项目根目录，Cursor 版本 >= 0.42

Q: MCP 返回的数据太旧？
A: 项目结构变化后需要重启 Cursor，MCP Server 会重新扫描

---
> Source: [yxc-gdut/second-brain-](https://github.com/yxc-gdut/second-brain-) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
