## warp2api-full

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## ⚠️ 重要注意事项 - 2025-09-24 更新

### Git Diff 单文件约束
**重要**: 使用 `git diff` 或类似命令查看文件更改时，**必须每次只检查一个文件**，避免执行问题。

✅ **正确示例**:
```bash
git diff file1.py
git diff file2.py
```

❌ **错误示例**:
```bash
git diff file1.py file2.py  # 这会导致执行失败！
```

### 已知问题和解决方案
- **空 content 错误**: 已修复 - 工具调用展开时使用空字符串而非 None
- **工具调用序列不完整**: 已添加自动清理逻辑
- **长文本响应中断**: 已实现智能分段（每段 1000 字符）
- **上下文重置**: 会自动提供任务延续提示

详细故障排查请参考：[docs/TROUBLESHOOTING.md](docs/TROUBLESHOOTING.md)

## 执行原则（强制）

- 读→改→再读验证（Read→Edit→Read）：进行任何文件修改前，必须先 Read 获取最新内容；完成 Edit 后，必须再次 Read 验证变更已生效且未误伤其它位置。
- 单点精确替换：使用唯一且完整的旧片段作为锚点进行替换，避免大范围模糊替换导致串改。
- 失败即复核：遇到 Edit 失败或不匹配，先 Read 核对真实文件内容，再调整旧片段后重试，直至验证通过。

### 文件更新操作提示词与规范（强制）

- 工具约束与常见原因
  - Edit 失败的常见原因：
    1) old_string 与文件内容不完全一致（换行/空格/缩进/标点差异）
    2) old_string 在文件中出现多处（非唯一），需要提供更大上下文或启用 replace_all
    3) 先前未 Read，导致基于过期上下文构造的 old_string
  - Edit 工具要求：
    - old_string 必须与文件中目标文本逐字符匹配（不含行号前缀）
    - 默认只替换首个匹配；若匹配不唯一且需全部替换，应设置 replace_all=true

- 推荐操作流程
  1) Read 目标文件，定位精确锚点（尽量包含上下文以确保唯一性）
  2) 构造 Edit：
     - old_string：粘贴文件中“完整且唯一”的原始文本片段（保持缩进与空白）
     - new_string：仅替换需要的内容，其余保持不动
     - 如需多处同样替换，使用 replace_all=true
  3) Read 再次验证变更：对目标行前后各读取若干行，确认已生效且未误伤

- 提示词模板
  - 单点替换（推荐）：
    「我将先 Read 文件，随后用 Edit 以‘唯一锚点片段’进行精确替换，完成后再 Read 验证。」
  - 批量替换：
    「该文本在文件中出现 N 处，需要全量替换，我将使用 replace_all=true，并在完成后 Read 检查所有位置。」
  - 失败处理：
    「Edit 失败/不匹配。我将重新 Read 获取最新内容，调整 old_string 的上下文后重试，直至验证通过。」

## 项目概述

Warp2Api 是一个基于 Python 的桥接服务，为 Warp AI 服务提供多种 API 兼容性。该项目采用双服务器架构，通过 protobuf 与 Warp AI 服务通信，同时为客户端提供多种兼容的 API 接口。

**核心特点**：
- 无需预填 token，程序自动获取匿名访问令牌（50次调用额度）
- **双 API 支持**：同时支持 OpenAI Chat Completions API 和 Anthropic Messages API
- 支持 system prompt 和 function calling（通过 MCP 兼容）
- 采用"偷梁换柱"方式处理历史消息和工具调用
- **完整格式转换**：支持 OpenAI ↔ Anthropic 格式双向转换
- 支持多模态数据包（可进一步开发）
- **流式响应兼容**：支持两种 API 格式的流式响应

## 常用命令

### 开发环境启动
```bash
# 安装依赖（推荐使用 uv）
uv sync

# 启动 Protobuf 桥接服务器 (端口 8000) - 必须先启动
python server.py
# 或使用项目脚本
warp-server

# 启动 API 服务器 (端口 8010) - 支持 OpenAI 和 Anthropic 格式 - 需要等待"热身"完成
python openai_compat.py
# 注意：pyproject.toml 中的 warp-test 脚本配置有误，应该指向 openai_compat:main
```

### 健康检查
```bash
# 检查 Protobuf 桥接服务器
curl http://localhost:8000/healthz

# 检查 API 服务器
curl http://localhost:8010/healthz
```

### 测试 API
```bash
# 测试 OpenAI Chat Completions 端点
curl -X POST http://localhost:8010/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{
    "model": "claude-3-sonnet",
    "messages": [{"role": "user", "content": "Hello"}],
    "stream": true
  }'

# 测试 Anthropic Messages 端点
curl -X POST http://localhost:8010/v1/messages \
  -H "Content-Type: application/json" \
  -d '{
    "model": "claude-3-5-sonnet-20241022",
    "max_tokens": 1024,
    "messages": [{"role": "user", "content": "Hello"}],
    "stream": true
  }'
```

### 代码质量检查
```bash
# 项目使用 uv 管理依赖，暂无配置的 lint 或 test 命令
# 可以手动运行基本的 Python 语法检查
python -m py_compile server.py openai_compat.py

# 检查项目依赖
uv tree
```

## 架构概览

### 双服务器架构
项目采用分层架构设计，包含两个主要服务器：

1. **Protobuf 桥接服务器** (`server.py`, 端口 8000)
   - 处理与 Warp AI 服务的 protobuf 通信
   - 提供 JSON ↔ Protobuf 编解码服务
   - 包含 WebSocket 监控功能
   - 管理 JWT 认证和令牌刷新
   - 解析 `server_message_data` 字段（Base64URL → 结构化对象）

2. **多格式 API 服务器** (`openai_compat.py`, 端口 8010)
   - 提供 OpenAI Chat Completions API 兼容接口
   - **新增 Anthropic Messages API 兼容接口**
   - 处理客户端请求并转换为 protobuf 格式
   - 支持两种格式的流式响应 (SSE)
   - 实现消息重排序和格式转换
   - **双向格式转换**：OpenAI ↔ Anthropic 自动转换
   - 启动时需要"热身"请求

### 核心模块结构

#### `protobuf2openai/` - 多格式 API 兼容层
- `app.py` - FastAPI 应用主入口，包含启动健康检查
- `router.py` - API 路由定义，处理 `/v1/chat/completions` 和 `/v1/messages` 端点
- `models.py` - Pydantic 数据模型，定义 OpenAI 和 Anthropic API 兼容的数据结构
- **`anthropic_converter.py` - Anthropic ↔ OpenAI 格式转换器**
- **`anthropic_sse_transform.py` - Anthropic 格式流式响应处理器**
- `bridge.py` - 桥接逻辑，连接多格式 API 和 Protobuf 服务
- `sse_transform.py` - OpenAI 格式服务器发送事件 (SSE) 流式响应处理
- `reorder.py` - 消息重排序逻辑，适配 Anthropic 风格对话
- `config.py` - 配置管理，读取环境变量和设置
- `packets.py` - 核心转换逻辑，处理消息格式转换
- `helpers.py` - 辅助函数，处理内容格式化

#### `warp2protobuf/` - Warp Protobuf 通信层
- `api/` - API 路由和处理逻辑
- `core/` - 核心功能模块
  - `auth.py` - JWT 认证管理，包括匿名令牌获取、刷新和验证
  - `protobuf_utils.py` - Protobuf 编解码工具函数
  - `logging.py` - 日志配置和管理
  - `schema_sanitizer.py` - 输入模式清理和验证
  - `session.py` - 会话管理
  - `protobuf.py` - Protobuf 运行时管理
  - `server_message_data.py` - server_message_data 字段解析
- `config/` - 配置文件和设置
- `warp/` - Warp 特定的通信逻辑

### 数据流架构

```
客户端应用 → 多格式 API 服务器 → Protobuf 桥接服务器 → Warp AI 服务
(8010端口)      (FastAPI)          (Protobuf处理)      (Warp服务)
                   ↓                     ↓
    OpenAI/Anthropic 格式 ←→ Protobuf JSON格式 ←→ Protobuf二进制
                   ↕
            格式自动转换层
```

## 关键技术细节

### 认证机制
- **匿名访问**: 程序自动获取匿名账号令牌（50次调用额度）
- **JWT 管理**: 自动令牌验证和刷新
- **令牌获取**: 通过匿名接口获取鉴权 token 和 refresh_token
- **配置**: 可选环境变量 `WARP_JWT` 和 `WARP_REFRESH_TOKEN`
- **实现**: 认证逻辑在 `warp2protobuf/core/auth.py` 中实现

### 消息处理流程
1. **客户端请求**: 发送 OpenAI 格式请求到 API 服务器
2. **消息重排序**: 验证请求格式并进行消息重排序（适配 Anthropic 风格）
3. **格式转换**: 请求转换为 protobuf JSON 格式
4. **Protobuf 编码**: 转换为 protobuf 二进制发送到桥接服务器
5. **Warp 通信**: 桥接服务器与 Warp AI 服务通信
6. **响应返回**: 通过相同路径返回给客户端

### Server Tool Call 处理
- **问题**: Warp 要求 task_context 中的消息必须以 server tool call 开头
- **解决方案**: 
  1. 发送初始请求获取 task_id 和 server tool call id
  2. 将 server tool call 塞到消息历史最前面
  3. 实现"偷梁换柱"，可以添加任意 task context
- **实现**: 在 `protobuf2openai/packets.py` 中的 `map_history_to_warp_messages` 函数

### Tool Call 支持
- **实现方式**: 通过 MCP (Model Context Protocol) 包装工具调用
- **转换**: 将 OpenAI tools 转换为 Warp mcp_context.tools
- **注意事项**:
  - 清理空的工具字段（如 `{}`），避免 Warp 忽略工具
  - 避免与自带工具重叠，防止服务端卡 bug
  - 添加约束禁止调用内部工具（如 read_files 等）

### System Message 支持
- **实现方式**: 在最后一个 user input 中作为附件文本添加
- **原因**: Warp 的 Rule 实现较为复杂，需要上传到服务器作为 memory
- **优势**: 利用注意力机制，实现效果良好
- **实现**: 在 `protobuf2openai/packets.py` 中的 `attach_user_and_tools_to_inputs` 函数

### 流式响应
- **协议**: 使用 Server-Sent Events (SSE) 实现流式响应
- **双格式兼容**: 完全兼容 OpenAI 和 Anthropic 的流式响应格式
- **OpenAI 处理**: 在 `protobuf2openai/sse_transform.py` 中处理转换逻辑
- **Anthropic 处理**: 在 `protobuf2openai/anthropic_sse_transform.py` 中处理转换逻辑
- **特性**: 支持两种格式的工具调用流式响应
- **自动转换**: 根据请求格式自动选择对应的响应格式

## 环境配置

### 必需环境变量
- 无需预填，程序自动获取匿名令牌

### 可选配置
- `WARP_JWT` - Warp 认证 JWT 令牌（可选）
- `WARP_REFRESH_TOKEN` - JWT 刷新令牌（可选）
- `HOST` - 服务器主机地址 (默认: `127.0.0.1`)
- `PORT` - OpenAI API 服务器端口 (默认: `8010`)
- `BRIDGE_BASE_URL` - Protobuf 桥接服务器 URL (默认: `http://localhost:8000`)

### 依赖管理
项目使用 `uv` 进行依赖管理，主要依赖包括：
- FastAPI - Web 框架
- Uvicorn - ASGI 服务器
- HTTPx - HTTP 客户端
- Protobuf - Protocol buffer 支持
- WebSockets - WebSocket 通信
- OpenAI - 类型兼容性

## 开发注意事项

### 服务器启动顺序
1. **必须先启动** Protobuf 桥接服务器 (`server.py`)
2. **再启动** OpenAI API 服务器 (`openai_compat.py`)
3. **等待热身**: OpenAI 服务器启动后需要等待"热身"请求完成
4. **依赖关系**: 后者依赖前者的健康检查

### 调试和开发
- **研究模式**: 可以连接 server.py 的 8000 端点进行编解码研究
- **日志记录**: 两个服务器都提供详细的日志记录
- **WebSocket 监控**: Protobuf 桥接服务器包含 WebSocket 监控端点 (`/ws`)
- **健康检查**: 使用 `/healthz` 端点检查服务状态

### 已知问题和限制
- **令牌刷新**: 刷新后可能出现 internal error，但通常很快恢复
- **工具重叠**: 与自带工具重叠时可能卡 bug
- **上下文限制**: 输入上下文过多时可能出现异常响应
- **上游限制**: 无法完全自定义工具，上游可能有预填提示词

### 错误处理
- **认证错误**: 触发自动令牌刷新
- **网络错误**: 包含重试逻辑
- **工具验证**: 清理无效工具字段
- **详细日志**: 错误日志记录在各自模块中

## 扩展潜力

### 多模态支持
- **现状**: Warp 数据包支持多模态，但当前实现未包含
- **开发**: 可通过扩展 protobuf 定义添加图片输入支持
- **实现**: 参考 `proto/` 目录下的 protobuf 定义

### 功能增强
- **更多工具类型**: 支持 Warp 的其他工具类型
- **自定义工具**: 改进工具自定义能力
- **性能优化**: 优化大上下文处理
- **部署版本**: 可考虑 Deno、Cloudflare 等多平台部署

### API 格式支持
- **OpenAI 兼容**: 完全支持 OpenAI Chat Completions API 格式和行为
- **Anthropic 兼容**: 完全支持 Anthropic Messages API 格式和行为
- **自动转换**: 内部自动处理两种格式之间的转换
- **流式响应**: 两种格式都支持标准的 SSE 流式响应
- **工具调用**: 两种格式都支持 function calling/tool use

### MCP 集成
- **通用方案**: 此 MCP 包装工具调用的方法可应用于其他项目
- **适用场景**: 如 claude2api、cursor2api 等代码助手的工具调用支持
- **优势**: 为免费账号提供 function calling 能力
- **双格式支持**: OpenAI tools 和 Anthropic tools 都通过 MCP 实现

## 相关文档

- **Function Call 转换机制**: `docs/function-call-tool-use-conversion.md`
- **Protobuf 定义**: `proto/` 目录
- **API 格式转换**: 详见 `protobuf2openai/anthropic_converter.py` 和相关转换器
- **使用示例**: 参考 README.md 中的使用说明
- **测试文件**: `test_anthropic_endpoint.py` 和 `test_anthropic_streaming.py`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/mucsbr) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
