## openapi2mcpserver

> export BASE_URL="https://api.example.com"

# OpenAPI to MCP Server 快速参考

## 快速启动

### 本地开发
```bash
# 设置环境变量
export BASE_URL="https://api.example.com"
export OPEN_API_DOC_JSON_URL="https://api.example.com/openapi.json"

# 运行服务器
python server.py
```

### 生产部署
```bash
uvx --from openapi2mcpserver
```

## 关键文件

- 主服务器: [server.py](mdc:server.py)
- 配置管理: [openapi2mcpserver/app_config.py](mdc:openapi2mcpserver/app_config.py)
- 项目配置: [pyproject.toml](mdc:pyproject.toml)
- 包信息: [openapi2mcpserver/__init__.py](mdc:openapi2mcpserver/__init__.py)

## 常见任务

### 修改版本号
1. 更新 [pyproject.toml](mdc:pyproject.toml) 中的 `version`
2. 更新 [openapi2mcpserver/__init__.py](mdc:openapi2mcpserver/__init__.py) 中的 `__version__`

### 添加新的配置项
在 [openapi2mcpserver/app_config.py](mdc:openapi2mcpserver/app_config.py) 的 `Config` 类中添加新属性

### 修改路由过滤逻辑
编辑 [server.py](mdc:server.py) 中的 `route_map_list` 构建逻辑

### 调试连接问题
1. 检查 `BASE_URL` 是否可访问
2. 验证 `OPEN_API_DOC_JSON_URL` 返回有效的 OpenAPI 3.0 JSON
3. 确认端口 9087 未被占用
4. 检查 FastMCP 版本兼容性（需要 >=2.8.0）

## 环境变量速查

| 变量名 | 必需 | 说明 |
|--------|------|------|
| `BASE_URL` | ✅ | API 服务器基础 URL |
| `OPEN_API_DOC_JSON_URL` | ✅ | OpenAPI 文档 JSON URL |
| `SERVER_NAME` | ❌ | MCP 服务器名称 |
| `ROUTE_MAPS` | ❌ | 路由过滤配置（JSON） |

## 故障排查

### 服务器启动失败
- 检查环境变量是否设置
- 验证 OpenAPI 文档 URL 是否可访问
- 确认依赖是否正确安装

### MCP 客户端连接失败
- 确认服务器正在运行（端口 9087）
- 检查客户端配置中的环境变量
- 验证 `uvx` 命令是否可用
- 确认传输协议配置正确（HTTP vs stdio）
- 检查 MCP 客户端是否支持 `streamable-http` 传输

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jeweis) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
