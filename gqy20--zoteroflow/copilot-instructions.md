## zoteroflow

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Task Master AI Instructions
**Import Task Master's development workflow commands and guidelines, treat as if import is in the main CLAUDE.md file.**
@./.taskmaster/CLAUDE.md

## 项目概述

ZoteroFlow2 是一个基于 Go 语言开发的智能文献管理工具，集成了 Zotero 数据库访问、MinerU PDF 解析、AI 智能分析和 MCP (Model Context Protocol) 服务器功能。项目包含约 6300 行 Go 代码，提供完整的学术文献处理流水线。

## 常用开发命令

### 构建和运行
```bash
# 进入服务器目录
cd server

# 标准构建（含UPX压缩）
make build

# 生产构建（功能完整且最小）
make build-prod

# 开发构建（保留调试信息）
make build-dev

# 运行主程序
make run

# 直接运行开发版本
make dev

# 查看CLI帮助
./bin/zoteroflow2 help
```

### 测试命令
```bash
# 基础单元测试
make test

# 带覆盖率的测试
make test-coverage

# MinerU集成测试
go run tests/test_mineru.go

# MCP基础功能测试
go run tests/mcp/test_mcp_basic.go

# Python集成测试
cd tests && python3 test_article_mcp.py
```

### 代码质量检查
```bash
# 快速检查（格式化 + vet）
make quick

# 完整检查（格式化 + lint + 测试）
make check

# 单独检查
make fmt    # 格式化代码
make lint   # golangci-lint检查
make vet    # go vet检查
```

### 项目状态检查
```bash
# 全面项目状态检查
go run tools/MCP_STATUS_CHECK.go

# MCP服务器功能测试（在server目录）
./bin/zoteroflow2 mcp

# Article MCP独立运行
uvx article-mcp server

# CLI功能测试
./bin/zoteroflow2 list
./bin/zoteroflow2 chat "测试问题"
```

## 核心架构

### 主要组件

1. **CLI应用程序** (`server/main.go` - 877行) - 命令行界面和核心功能集成
   - 文献管理命令（list, search, open, doi）
   - AI助手对话功能（chat）
   - 智能文献分析（related）
   - MCP服务器模式

2. **ZoteroDB** (`server/core/zotero.go` - 783行) - Zotero SQLite 数据库访问层
   - 只读模式访问 Zotero 数据库
   - 解析 PDF 元数据和文件路径
   - 处理 Zotero 存储系统格式 (storage:XXXXXX.pdf)

3. **MinerUClient** (`server/core/mineru.go` - 715行) - MinerU PDF 解析 API 客户端
   - 支持单文件和批量处理
   - 处理文件上传和结果轮询
   - 使用命名类型而非匿名结构体

4. **AIClient** (`server/core/ai.go` - 706行) - AI 模型集成
   - 支持智谱 GLM-4.6 模型
   - 集成学术文献分析功能
   - 异步处理和错误恢复

5. **Organizer** (`server/core/organizer.go` - 673行) - 数据组织和处理
   - 文献数据结构管理
   - CSV文件生成和处理
   - 数据验证和清理

6. **AI-MCP Bridge** (`server/mcp/ai_mcp_bridge.go` - 942行) - AI与MCP协议桥接
   - AI功能的MCP工具封装
   - 智能对话和分析接口
   - 上下文管理和会话处理

7. **MCPManager** (`server/mcp/mcp_manager.go` - 517行) - MCP 服务器管理器
   - 配置驱动的 MCP 服务器集成
   - 支持本地和外部 MCP 服务器
   - JSON-RPC 2.0 协议实现

8. **RelatedLiterature** (`server/mcp/related_literature.go` - 354行) - 相关文献分析
   - 基于 AI 的文献关联分析
   - 支持多种学术数据库搜索

### 配置管理

配置优先级（从高到低）：
1. MCP 客户端环境变量
2. 项目 `.env` 文件
3. 代码默认值

关键配置项：
```bash
# Zotero 配置
ZOTERO_DB_PATH=/path/to/zotero.sqlite
ZOTERO_DATA_DIR=/path/to/zotero/storage

# AI 配置
AI_API_KEY=your_api_key
AI_BASE_URL=https://open.bigmodel.cn/api/coding/paas/v4
AI_MODEL=glm-4.6

# MinerU 配置
MINERU_API_URL=https://mineru.net/api/v4
MINERU_TOKEN=your_token

# 目录配置
RESULTS_DIR=data/results
RECORDS_DIR=data/records
CACHE_DIR=~/.zoteroflow/cache
```

### CLI 功能模块

#### 文献管理
- `list` - 列出所有已解析的文献
- `search <关键词>` - 按标题搜索并解析文献
- `open <名称>` - 打开指定文献的文件夹
- `doi <DOI号>` - 按DOI搜索并解析文献

#### AI助手对话
- `chat` - 进入交互式AI对话模式
- `chat <问题>` - 单次AI问答
- `chat --doc=文献名 <问题>` - 基于指定文献的AI对话

#### 智能文献分析
- `related <文献名/DOI> <问题>` - 查找相关文献并AI分析

### 数据流程

1. **配置加载**：从 `.env` 和环境变量加载配置
2. **数据库连接**：以只读模式连接 Zotero 数据库
3. **PDF 发现**：从 Zotero 存储系统发现 PDF 文件
4. **PDF 解析**：上传至 MinerU 进行解析
5. **结果存储**：将解析结果存储到 `RESULTS_DIR` 并生成CSV记录
6. **AI 分析**：使用 AI 模型进行智能分析
7. **MCP 服务**：通过 MCP 协议暴露功能

### 目录结构

```
server/
├── main.go                    # CLI应用程序入口
├── config/config.go           # 配置管理
├── core/                      # 核心功能模块
│   ├── zotero.go             # Zotero数据库访问
│   ├── mineru.go             # MinerU PDF解析
│   ├── ai.go                 # AI模型集成
│   ├── parser.go             # PDF解析器
│   └── organizer.go          # 数据组织
├── mcp/                       # MCP服务器模块
│   ├── mcp_manager.go        # MCP管理器
│   ├── ai_mcp_bridge.go      # AI-MCP桥接
│   └── related_literature.go # 相关文献分析
└── tests/                     # 测试文件
    └── test_mineru.go        # MinerU集成测试
```

## MCP 集成

### 本地 MCP 工具

项目内置 6 个本地 MCP 工具：
- `list_literature` - 列出文献
- `search_literature` - 搜索文献
- `get_literature_details` - 获取文献详情
- `parse_pdf` - 解析 PDF
- `analyze_literature` - AI 分析文献
- `find_related_literature` - 查找相关文献

### 外部 MCP 集成

支持 Article MCP 服务器集成，提供 10 个学术工具：
- Europe PMC 文献搜索
- arXiv 预印本搜索
- 文献详细信息获取
- 参考文献管理
- 期刊质量评估
- 文献关联分析

### Claude Desktop 配置

最小配置示例：
```json
{
  "mcpServers": {
    "zoteroflow2": {
      "command": "/home/qy113/workspace/note/zo/ZoteroFlow2/server/bin/zoteroflow2",
      "args": ["mcp"],
      "env": {
        "ZOTERO_DB_PATH": "/home/qy113/workspace/note/zo/zotero_file/zotero.sqlite",
        "ZOTERO_DATA_DIR": "/home/qy113/workspace/note/zo/articles"
      }
    }
  }
}
```

## 测试框架

### 测试目录结构

```
tests/
├── test_mineru.go         # MinerU API 集成测试
└── mcp/                   # MCP 协议测试
    └── test_mcp_basic.go  # MCP 基础功能测试
```

### 测试覆盖范围

- ✅ MinerU PDF 解析 API 集成测试
- ✅ MCP 协议通信（JSON-RPC 2.0）
- ✅ Article MCP 服务器连接
- ✅ Go MCP 客户端实现
- ✅ AI 智能分析集成
- ✅ 多数据库文献搜索
- ✅ 文献质量评估
- ✅ 配置文件管理

**注意**: 目前没有传统的单元测试文件，主要通过集成测试验证功能。

## 开发工具

### 状态检查工具

`tools/MCP_STATUS_CHECK.go` - 全面的项目状态检查工具：
- 项目结构完整性检查
- 编译状态验证
- MCP 服务器功能测试
- 外部 MCP 集成验证

### 依赖管理

```bash
# 下载依赖
make deps

# 升级依赖
make mod-upgrade

# 整理依赖
go mod tidy
```

## 关键文件说明

### 服务器核心文件
- `server/main.go` - CLI应用程序入口（877行）
- `server/config/config.go` - 配置管理模块
- `server/core/` - 核心功能模块目录
- `server/mcp/` - MCP服务器模块目录
- `server/tests/` - 集成测试文件

### 配置文件
- `.env` - 根目录环境变量配置
- `server/.env` - 服务器目录环境变量配置（推荐使用）
- `server/mcp_config.json` - MCP 服务器配置
- `server/external-mcp-servers.json` - 外部 MCP 服务器配置

### 工具文件
- `tools/MCP_STATUS_CHECK.go` - 项目状态检查工具
- `server/Makefile` - 构建和开发命令

### 文档
- `docs/IMPLEMENTATION.md` - 技术实现方案
- `docs/UI_IMPLEMENTATION_PLAN.md` - UI实现计划
- `docs/think.md` - 设计理念

## 开发最佳实践

### 开发流程

1. **开发前**：运行 `go run tools/MCP_STATUS_CHECK.go` 检查状态
2. **编码后**：运行 `make quick` 进行快速检查
3. **提交前**：运行 `make check` 进行完整检查
4. **发布前**：运行完整测试套件

### 代码规范

- 使用英文日志输出
- 优先使用命名类型而非匿名结构体
- 遵循约定式提交格式：`<type>(<scope>): <description>`
- 代码覆盖率要求：80%+（目前主要通过集成测试实现）

### 调试技巧

- 使用 `make build-dev` 构建开发版本进行调试（保留调试信息）
- CLI 问题：运行 `./bin/zoteroflow2 help` 查看可用命令
- MCP 连接问题：检查 `server/mcp_config.json` 配置
- AI 集成问题：验证 `server/.env` 文件中的 AI 配置
- PDF 解析问题：确认 MinerU API 令牌有效性
- Zotero 数据库问题：确认数据库路径和存储目录正确

## 常见问题解决

### 编译问题
- 确保使用 Go 1.21+
- 运行 `make deps` 更新依赖
- 检查 `go.mod` 和 `go.sum` 文件

### MCP 连接问题
- 验证二进制文件路径：`ls -la server/bin/`
- 检查环境变量配置
- 运行 `go run tools/MCP_STATUS_CHECK.go` 诊断
- 测试 MCP 服务器：`./server/bin/zoteroflow2 mcp`

### AI 集成问题
- 检查 `server/.env` 文件中的 AI 配置
- 验证 API 密钥和模型名称
- 确认网络连接和 API 端点可访问性
- 测试 AI 功能：`./server/bin/zoteroflow2 chat "测试问题"`

### Zotero 数据库问题
- 确认 Zotero 数据库路径正确（在 `server/.env` 中设置 `ZOTERO_DB_PATH`）
- 检查文件权限（只读访问）
- 验证 Zotero 存储目录路径（`ZOTERO_DATA_DIR`）

### 构建问题
- 确保在 `server` 目录下运行 make 命令
- 检查 Go 版本（需要 1.21+）
- 运行 `make deps` 更新依赖

## 部署注意事项

### 生产环境构建
```bash
cd server
make build-prod
```

### 环境变量配置
确保生产环境中正确设置所有必需的环境变量，特别是：
- `ZOTERO_DB_PATH`
- `ZOTERO_DATA_DIR`
- `AI_API_KEY`
- `MINERU_TOKEN`

### 权限要求
- Zotero 数据库：只读访问权限
- 缓存目录：读写权限
- 日志目录：写入权限

## 项目状态

### 当前状态
- ✅ 完整的 CLI 应用程序实现
- ✅ Zotero 数据库集成
- ✅ MinerU PDF 解析集成
- ✅ AI 智能分析功能
- ✅ MCP 服务器实现
- ✅ 相关文献分析功能
- 🔄 Web UI 开发中（见 `docs/UI_IMPLEMENTATION_PLAN.md`）

### 技术指标
- **代码量**: 约 6300 行 Go 代码
- **构建时间**: < 30 秒
- **二进制大小**: ~7MB（UPX 压缩后）
- **支持平台**: Linux/macOS

---

**文档版本**: v2.1
**创建日期**: 2025-10-04
**最后更新**: 2025-10-04
**维护者**: ZoteroFlow2 开发团队
**更新频率**: 根据项目变化更新

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/gqy20)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/gqy20)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
