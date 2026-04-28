## unifiles-mcp

> Python MCP 官方 SDK (FastMCP) 代码风格指南


# Python MCP SDK (FastMCP) Code Style

> **Context**: 适用于使用 `mcp` 官方 SDK 的 `FastMCP` 高级接口开发服务器。
> **Focus**: 使用装饰器、类型提示自动生成 Schema、简化生命周期管理。
> **Reference**: https://pypi.org/project/mcp/

## 1. 核心原则 (Core Principles)

- **FastMCP 优先**: 始终使用 `mcp.server.fastmcp.FastMCP` 类，避免直接操作底层的 `Server` 类。
- **类型驱动**: 利用 Python 类型提示 (`str`, `int`, `bool`, `list`, 等) 自动生成 MCP Schema；**优先使用简单类型平铺传参**，避免用单个字典/对象包住所有参数。
- **文档即描述**: 函数的 Docstring 会自动转换为工具/资源的描述字段。
- **上下文感知**: 使用 `Context` 对象处理日志、进度报告和二进制数据。

## 2. 基础架构 (Basic Structure)

标准的 FastMCP 服务器结构如下：

```python
from mcp.server.fastmcp import FastMCP, Context, Image

# 1. 初始化服务器
mcp = FastMCP("My Server")

# 2. 定义工具 (Tools)
@mcp.tool()
async def calculate_metrics(data: list[float], threshold: float = 0.5) -> dict[str, float]:
    """
    计算数据指标。
    
    Args:
        data: 输入的数据列表
        threshold: 过滤阈值
    """
    filtered = [x for x in data if x > threshold]
    return {
        "count": len(filtered),
        "average": sum(filtered) / len(filtered) if filtered else 0
    }

# 3. 定义资源 (Resources)
@mcp.resource("file://{path}")
def read_custom_file(path: str) -> str:
    """读取自定义格式文件"""
    return f"Mock content for {path}"

# 4. 运行入口
if __name__ == "__main__":
    mcp.run()
```

## 3. 关键模式 (Key Patterns)

### 3.1 工具传参：简单类型平铺 (Simple Flat Parameters)
**优先使用简单类型平铺传参**，不要用单个 Pydantic 模型或字典包住所有参数。这样 MCP 客户端只需传 `key: value`，无需构造 `params: { ... }` 对象。校验在函数内部完成。

- 必填/常用参数：`str`、`int`、`bool`、`list`、`tuple` 等，用 `Annotated[T, Field(description="...")]` 写描述。
- 可选参数：带默认值即可，如 `limit: int = 10`。
- 仅在确有复杂嵌套、且多工具复用时，再考虑 Pydantic 模型。

```python
from typing import Annotated
from pydantic import Field

@mcp.tool()
async def search_database(
    query: Annotated[str, Field(description="搜索关键词")],
    limit: Annotated[int, Field(description="最多返回条数")] = 10,
    tags: list[str] | None = None,
    ctx: Context,
) -> list[str]:
    """在数据库中搜索内容。"""
    if tags is None:
        tags = []
    # 校验在内部完成，如 limit 范围等
    return [f"Result for {query}"]
```

### 3.2 上下文交互 (Context Interaction)
当需要发送日志、报告进度或访问元数据时，在参数中添加 `ctx: Context`。

```python
@mcp.tool()
async def process_large_file(filename: str, ctx: Context) -> str:
    await ctx.info(f"Starting processing for {filename}")
    
    for i in range(10):
        # 模拟耗时操作
        progress = (i + 1) / 10
        await ctx.report_progress(progress=progress, total=1.0, message=f"Step {i+1}/10")
        
    await ctx.info("Processing complete")
    return "Done"
```

### 3.3 返回多媒体内容 (Multimedia Content)
FastMCP 允许直接返回 `Image` 对象或文本，它会自动处理格式包装。

```python
@mcp.tool()
async def generate_chart(data: list[int]) -> Image:
    # 假设这里生成了图片字节流
    img_data = b"..." 
    return Image(data=img_data, format="png")
```

### 3.4 提示词模板 (Prompts)
定义 LLM 可以使用的预设提示词。

```python
@mcp.prompt()
def debug_error(error_log: str) -> str:
    """分析错误日志的提示词模板"""
    return f"Please analyze the following error log and suggest fixes:\n\n{error_log}"
```

## 4. 最佳实践 (Best Practices)

- **Docstring 规范**: 保持 Docstring 清晰简洁，因为这是 LLM 理解工具用途的唯一途径。
- **避免 Print**: 永远不要使用 `print()`，这会破坏 MCP 协议的 stdio 通信。使用 `await ctx.info()`、`await ctx.error()` 或 `logging` (输出到 stderr)。
- **异步 I/O**: 涉及网络请求或文件操作时，始终使用 `async def`。
- **Context 方法调用**: Context 的所有方法（`info`, `error`, `warning`, `debug`, `report_progress` 等）都是异步的，必须使用 `await`。

## 5. 常见错误 (Common Pitfalls)

- **参数无类型注解**: `def my_tool(x):` ❌ (FastMCP 无法生成 Schema)
- **参数有类型注解**: `def my_tool(x: int):` ✅
- **用对象包住所有参数**: `def my_tool(params: SearchParams):` ❌ 增加客户端调用复杂度；应改为平铺参数 `def my_tool(query: str, limit: int = 10):` ✅
- **混淆 Context 位置**: `Context` 参数可以放在任意位置，FastMCP 会自动注入，不需要客户端传递。
- **忘记 await**: Context 方法都是异步的，必须使用 `await ctx.info()` 而不是 `ctx.info()` ❌
- **装饰器语法**: 官方 SDK 推荐使用 `@mcp.tool()`（带括号），虽然不带括号也支持，但带括号更明确

## 6. 官方文档参考

- **MCP Python SDK 官方文档**: https://pypi.org/project/mcp/
- **API 参考**: https://modelcontextprotocol.github.io/python-sdk/

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Asheng008) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
