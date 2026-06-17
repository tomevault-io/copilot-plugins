## traceweaver

> TraceWeaver 是一个 LLM-first 的可插拔网络流量诊断平台。用户上传 PCAP 文件，系统通过 tshark 解析 → profile enricher 语义增强 → 工具层查询 → LLM 多轮推理，输出结构化诊断报告。

# CLAUDE.md — TraceWeaver 开发指南

## 项目简介

TraceWeaver 是一个 LLM-first 的可插拔网络流量诊断平台。用户上传 PCAP 文件，系统通过 tshark 解析 → profile enricher 语义增强 → 工具层查询 → LLM 多轮推理，输出结构化诊断报告。

核心创新：**Profile 机制**——新增协议只需写一个 profile 包，不改 core 代码。

## 常用命令

```bash
# 安装（开发模式）
pip install -e ".[dev,mcp-server]"

# 运行全部测试
pytest tests/ -v

# 运行单个模块测试
pytest tests/serve/ -v
pytest tests/profiles/open5gs_5gc/ -v

# CLI 诊断
traceweaver analyze --profile open5gs_5gc --pcap <path> "问题描述"

# MCP 服务
traceweaver serve --profile open5gs_5gc --profile web_l4l7_failures

# 列出已安装的 profile
traceweaver profile list
```

## 架构概览

```
traceweaver/
├── core/                  # 协议层（不依赖任何具体 profile）
│   ├── protocols.py       # ABC 定义：Record, Source, SourceHandle, Tool, ToolSpec, ToolContext, ToolResult
│   ├── kernel.py          # Agent 循环引擎（多轮工具调用 + LLM 推理）
│   ├── profile/           # Profile 加载（yaml_loader, entry_points, 三源 loader）
│   ├── tools/             # ToolRegistry（注册、分发、结果合约检查）
│   ├── source/            # SourceRegistry + EnrichedHandle
│   └── knowledge/         # 知识库（Markdown 文件搜索）
├── builtin/               # 内置实现
│   ├── sources/pcap.py    # PcapSource（tshark 解析）
│   └── tools/             # 3 个内置工具：query_records, get_records_around, search_knowledge
├── profiles/              # 内置 profile
│   ├── open5gs_5gc/       # 5G 核心网诊断
│   └── web_l4l7_failures/ # DNS/TCP/TLS/Web 诊断
├── serve/                 # MCP 服务层
│   ├── runtime.py         # ServeContext, MultiServeContext, LoadCaptureTool
│   ├── mcp.py             # build_mcp_server / build_mcp_server_multi
│   └── session.py         # CaptureSession（可变 source handle 包装）
├── cli/                   # 命令行入口
│   ├── analyze.py         # traceweaver analyze
│   ├── serve.py           # traceweaver serve
│   ├── replay.py          # traceweaver replay
│   └── profile.py         # traceweaver profile list
└── recording/             # LLM 会话录制/回放
```

### 关键分层

- **core/** 只定义协议（ABC），不 import 任何 profile 代码
- **builtin/** 实现通用功能（pcap 解析 + 3 个内置工具）
- **profiles/** 每个 profile 是自包含的：`profile.yaml` + `enrich.py` + `fields.py` + `tools/` + `prompts/` + `knowledge/`
- **serve/** 把工具通过 MCP 暴露给外部 agent

### 数据流

```
pcap → PcapSource.ingest() → SourceHandle
     → Profile.enricher(SourceHandle) → EnrichedHandle（语义事件）
     → ToolRegistry.invoke(name, args, ToolContext) → ToolResult
     → Kernel 多轮循环（最多 12 轮）→ 结构化 JSON 诊断
```

## 核心约定

### 工具只返事实，不下判断

ToolResult.data 中**禁止**出现以下 key：`verdict`, `root_cause`, `failure_point`, `confidence`。ToolRegistry 会在 `invoke()` 时强制检查，违反直接抛异常。诊断判断属于 LLM，不属于工具。

### Profile 开发规范

一个 profile 包含：

| 文件 | 用途 |
|------|------|
| `profile.yaml` | 元数据 + source_config + display_filter + 字段列表 |
| `enrich.py` | `enrich(record) → dict` 纯函数，状态无关 |
| `fields.py` | 字段映射表（枚举值 → 可读名称） |
| `tools/*.py` | 每个工具是一个 `Tool` 子类，`spec` 用 `ToolSpec.from_pydantic()` 生成 |
| `prompts/system.md` | LLM 系统提示词 |
| `knowledge/*.md` | 可被 `search_knowledge` 检索的知识库 |

工具的 `parameters_schema` 推荐用 Pydantic 模型 + `ToolSpec.from_pydantic()` 自动生成：

```python
from pydantic import BaseModel
from traceweaver.core.protocols import ToolSpec

class _MyInput(BaseModel):
    flow_id: str
    limit: int = 50

class MyTool(Tool):
    spec = ToolSpec.from_pydantic(
        name="my_tool",
        description="...",
        model=_MyInput,
    )

    def run(self, ctx: ToolContext, **kwargs) -> ToolResult:
        args = _MyInput(**kwargs)
        # ... 使用 args.flow_id, args.limit
```

### CaptureSession 模式（MCP serve）

MCP 服务启动时不绑定 pcap。Agent 通过 `load_capture(path=...)` 动态加载。工具通过 `ctx.source_handle` 访问当前 capture，它是一个 `CaptureSession` 代理对象——`session.load()` 后所有后续工具调用自动看到新数据。

### 多 Profile 工具命名

多 profile 模式下，profile 专属工具自动加 `<profile>__` 前缀（如 `open5gs_5gc__summarize_capture`）。内置工具（`query_records`, `get_records_around`, `search_knowledge`）不加前缀，路由到默认 profile。

## 红线（不要做的事）

1. **不要在工具里做诊断判断**（见上方 "工具只返事实"）
2. **不要让 core/ 依赖某个 profile**（core import profiles 是架构违反）
3. **不要为单一 case 改 prompt 或加硬编码 hint**
4. **不要把 log/metric/trace 写进 builtin**（多源作为外部包扩展，不进 core）
5. **不要写"规则兜底 + LLM 补充"的混合诊断**
6. **不要保留 v1 兼容层**

## 测试

测试目录结构镜像源码结构：

```
tests/
├── core/                  # core 模块测试
│   ├── kernel/            # kernel 循环、遥测、schema 校验
│   ├── profile/           # loader、yaml_loader、runtime
│   ├── source/            # pcap slicing、enriched handle
│   ├── tools/             # registry、coercion、builtin 工具
│   └── replay/            # 录制/回放测试
├── profiles/              # profile 专属测试
│   ├── open5gs_5gc/       # enrich、fields、tools、profile 加载
│   └── web_l4l7_failures/ # enrich、tools
├── serve/                 # MCP serve 测试
│   ├── test_mcp.py        # 单元测试（mock server）
│   ├── test_mcp_e2e.py    # E2E 内存测试（真实 MCP 握手）
│   └── test_multi_profile.py
├── cli/                   # CLI 测试
└── fixtures/              # 测试数据
    ├── web_l4l7/          # docker-compose + canonical pcaps
    └── external_profile_pkg/  # 第三方 profile 包样板
```

Profile 测试通常使用 conftest.py 中的 fixture 加载真实 pcap 并执行 enrich，然后对 enriched records 做断言。

## LLM 集成

- 通过 LiteLLM 调用，支持任何 OpenAI 兼容 API
- `max_tokens=12288`, `timeout=300s`
- Kernel 有防重复调用、schema retry、budget warning 保护
- 推荐模型：MiniMax-M2.7（9/9 精度，$0.34/次）或同级；本地开发用 qwen3-14b（7/9，零成本）

## MCP 配置示例

```json
{
  "mcpServers": {
    "traceweaver": {
      "command": "traceweaver",
      "args": ["serve", "--profile", "open5gs_5gc", "--profile", "web_l4l7_failures"]
    }
  }
}
```

配置写一次即可。Agent 在对话中通过 `load_capture(path=...)` 加载抓包文件，换文件不用重启服务。

---
> Source: [xujiayuxian-png/TraceWeaver](https://github.com/xujiayuxian-png/TraceWeaver) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
