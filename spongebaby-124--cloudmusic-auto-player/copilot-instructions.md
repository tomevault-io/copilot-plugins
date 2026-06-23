## myrules

> 使用标准MCP项目结构，FastMCP为主要框架：

# MCP官方库开发 - Cursor Rules

## 项目结构
使用标准MCP项目结构，FastMCP为主要框架：
```
project/
├── src/
│   └── server.py          # 主服务器文件
├── config.json            # MCP配置
├── test_server.py         # 测试脚本
├── README.md              # 文档
└── pyproject.toml         # 依赖管理
```

## 核心开发模式

### 1. 服务器初始化
```python
from mcp.server.fastmcp import FastMCP

# 创建服务器实例
mcp = FastMCP("服务器名称")

# 运行服务器
if __name__ == "__main__":
    mcp.run()
```

### 2. 工具定义标准
```python
@mcp.tool()
def tool_name(param1: str, param2: bool = False) -> dict:
    """工具描述
    
    Args:
        param1: 参数1描述
        param2: 参数2描述，默认False
    """
    try:
        # 业务逻辑
        result = process_data(param1, param2)
        return {
            "success": True,
            "data": result,
            "message": "操作成功"
        }
    except Exception as e:
        return {
            "success": False,
            "error": str(e)
        }
```

### 3. 资源定义
```python
@mcp.resource("uri://pattern/{param}")
def resource_name(param: str) -> str:
    """资源描述"""
    return f"资源内容: {param}"
```

### 4. 提示定义
```python
@mcp.prompt()
def prompt_name(input_text: str) -> str:
    """提示描述"""
    return f"处理提示: {input_text}"
```

## 开发原则

### 代码规范
- 函数命名：snake_case
- 类型注解：必须提供
- 文档字符串：描述+参数说明
- 错误处理：统一返回格式

### 响应格式
统一使用以下响应格式：
```python
# 成功响应
{
    "success": True,
    "data": {...},          # 可选
    "message": "描述"       # 可选
}

# 错误响应
{
    "success": False,
    "error": "错误描述"
}
```

### 参数处理
- 必需参数：不提供默认值
- 可选参数：提供合理默认值
- 类型验证：依赖FastMCP自动处理
- 参数文档：在docstring中详细说明

## 测试规范

### 测试脚本模板
```python
#!/usr/bin/env python3
import subprocess
import sys
import time

def test_mcp_server():
    """测试MCP服务器"""
    print("🧪 开始测试MCP服务器...")
    
    process = subprocess.Popen([
        sys.executable, "src/server.py"
    ], stdin=subprocess.PIPE, stdout=subprocess.PIPE, 
      stderr=subprocess.PIPE, text=True)
    
    try:
        time.sleep(2)
        if process.poll() is None:
            print("✅ 服务器启动成功")
            time.sleep(3)
            return True
        else:
            print("❌ 服务器启动失败")
            return False
    finally:
        process.terminate()
        process.wait(timeout=3)

if __name__ == "__main__":
    success = test_mcp_server()
    sys.exit(0 if success else 1)
```

## 配置管理

### MCP配置文件
```json
{
  "mcpServers": {
    "server-name": {
      "command": "python",
      "args": ["path/to/server.py"],
      "env": {
        "KEY": "value"
      }
    }
  }
}
```

### 依赖管理
```toml
[project]
name = "mcp-server-name"
version = "1.0.0"
dependencies = [
    "mcp>=1.9.0"
]

[project.optional-dependencies]
dev = ["pytest", "black", "mypy"]

```

---
> Source: [SpongeBaby-124/CloudMusic_Auto_Player](https://github.com/SpongeBaby-124/CloudMusic_Auto_Player) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-23 -->
