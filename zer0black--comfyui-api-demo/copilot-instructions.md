## comfyui-api-demo

> 本文件为在 ComfUI-Api-Demo 代码库中工作的 AI 代理提供指导。

# ComfyUI API Demo - Agent 指南

本文件为在 ComfUI-Api-Demo 代码库中工作的 AI 代理提供指导。

## 项目概述

ComfyUI API 完整演示程序，涵盖工作流导出、API 调用流程、参数说明及 Python 代码实现。
- Python 版本: 3.13+
- 主要功能: 基础同步调用、WebSocket 实时监听、批量图像生成、图像下载、统一 API 客户端

## 构建和运行命令

采用如下指令启动main函数
`uv.exe run .venv/Scripts/python.exe main.py`

### 运行主程序

```bash
# 交互式模式（推荐新手）
uv.exe run .venv/Scripts/python.exe main.py

# 运行所有演示
uv.exe run .venv/Scripts/python.exe main.py --mode demo

# 单张生成（同步）
uv.exe run .venv/Scripts/python.exe main.py --mode sync

# 单张生成（WebSocket 实时）
uv.exe run .venv/Scripts/python.exe main.py --mode ws

# 批量生成
uv.exe run .venv/Scripts/python.exe main.py --mode batch

# 指定服务器地址
uv.exe run .venv/Scripts/python.exe main.py --server http://127.0.0.1:8188

# 指定工作流文件
uv.exe run .venv/Scripts/python.exe main.py --workflow config/workflow_api.json
```

### 依赖安装

```bash
uv install -r requirements.txt
```

依赖包括:
- `requests>=2.31.0` - HTTP 请求库
- `websocket-client>=1.6.0` - WebSocket 客户端

可选依赖:
- `Pillow` - 用于生成缩略图

### 测试

**注意**: 本项目目前没有测试文件。

如果需要运行模块单独测试:

```bash
# 测试基础同步调用
python comfyui/basic_sync.py

# 测试 WebSocket 客户端
python comfyui/websocket_client.py

# 测试统一客户端
python comfyui/comfyui_client.py

# 测试批量生成器
python comfyui/batch_generator.py

# 测试图像下载器
python comfyui/image_downloader.py
```

## 代码风格指南

### 命名约定

- **类名**: PascalCase (例如: `ComfyUISyncClient`, `BatchGenerator`)
- **函数/方法**: snake_case (例如: `generate_image`, `modify_parameter`)
- **变量**: snake_case (例如: `server_address`, `workflow_file`)
- **常量**: UPPER_SNAKE_CASE (例如: `SERVER_ADDRESS`, `DEFAULT_SAVE_DIR`)
- **私有方法**: 以下划线开头 (例如: `_load_workflow`, `_generate_sync`)

### 导入顺序

1. 标准库导入
2. 第三方库导入
3. 本地模块导入

示例:
```python
import argparse
import json
import time

import requests
import websocket

from comfyui.comfyui_client import ComfyUIUnifiedClient
import config
```

### 类型注解

使用 `typing` 模块进行类型注解:

```python
from typing import List, Dict, Optional, Callable

def generate_image(
    prompt_text: str,
    width: int = 768,
    height: int = 768,
    seed: int = -1,
) -> Dict:
    """生成图像"""
    pass

def batch_generate(
    prompts: List[str],
    download: bool = False,
) -> List[Dict]:
    """批量生成"""
    pass
```

### 文档字符串

使用中文文档字符串，格式:

```python
def generate_image(self, prompt_text: str, width: int = 768):
    """
    同步生成图像

    Args:
        prompt_text: 提示词
        width: 图像宽度

    Returns:
        dict: 包含生成结果的字典
    """
    pass
```

### 错误处理

- 使用明确的异常类型
- 在异常处理中提供清晰的错误消息

```python
try:
    response = requests.post(endpoint, json=payload)
    response.raise_for_status()
except requests.exceptions.RequestException as e:
    raise ConnectionError(f"请求失败: {e}")
except FileNotFoundError:
    raise FileNotFoundError(f"找不到工作流文件: {self.workflow_file}")
```

### 文件编码

始终使用 UTF-8 编码:

```python
with open("config/workflow.json", "r", encoding="utf-8") as f:
    workflow = json.load(f)
```

### 打印和日志

本项目使用 `print()` 进行输出，不使用 logging 模块。格式约定:

```python
print("[OK] 工作流加载成功")           # 成功消息
print("[ERROR] 执行错误")              # 错误消息
print("[STATUS] 状态更新")             # 状态消息
print("=" * 60)                        # 分隔线
```

### 配置管理

所有配置项集中在 `config/config.py` 文件中:

```python
# 服务器地址
SERVER_ADDRESS = "http://127.0.0.1:8188/api"

# 工作流文件路径
WORKFLOW_FILE = "config/workflow_api.json"

# 默认参数
DEFAULT_IMAGE_WIDTH = 768
DEFAULT_IMAGE_HEIGHT = 768
DEFAULT_STEPS = 50
DEFAULT_CFG = 7.5
```

在代码中使用配置:

```python
import config

self.server_address = server_address or config.SERVER_ADDRESS
self.workflow_file = workflow_file or config.WORKFLOW_FILE
```

### 路径处理

使用 `os.path` 处理文件路径，注意添加项目根目录到 Python 路径:

```python
import sys
import os

# 添加项目根目录到 Python 路径
sys.path.insert(0, os.path.abspath(os.path.join(os.path.dirname(__file__), "..")))
```

### 节点 ID 处理

ComfyUI 工作流使用节点 ID（如 "91", "86:3", "86:58"）来标识节点。常见节点:

- `"91"`: 提示词节点 (PrimitiveStringMultiline)
- `"86:3"`: KSampler 节点（控制种子、步数、CFG）
- `"86:58"`: Latent 图像节点（控制宽度、高度）
- `"86:81"`: CLIP Text Encode（正向提示词）
- `"86:7"`: CLIP Text Encode（负向提示词）

### 参数修改模式

```python
def modify_parameter(self, node_id: str, parameter_name: str, value):
    """修改工作流参数"""
    if node_id not in self.workflow:
        raise KeyError(f"节点ID不存在: {node_id}")

    if parameter_name not in self.workflow[node_id]["inputs"]:
        raise KeyError(f"参数名不存在: 节点{node_id}没有参数{parameter_name}")

    self.workflow[node_id]["inputs"][parameter_name] = value
```

### WebSocket 注意事项

- WebSocket 地址不需要 `http://` 前缀（使用 `ws://`）
- HTTP API 地址需要包含 `/api`（例如: `http://127.0.0.1:8188/api`）
- WebSocket 客户端使用 `client_id` 来标识连接

```python
# HTTP API
SERVER_ADDRESS = "http://127.0.0.1:8188/api"
endpoint = f"{config.SERVER_ADDRESS}/prompt"

# WebSocket
WEBSOCKET_ADDRESS = "127.0.0.1:8188"
ws_url = f"ws://{self.server_address}/ws?clientId={self.client_id}"
```

### JSON 处理

使用 `json.dump()` 和 `json.load()` 处理 JSON 文件:

```python
# 读取 JSON
with open("config/workflow.json", "r", encoding="utf-8") as f:
    workflow = json.load(f)

# 写入 JSON
with open("output.json", "w", encoding="utf-8") as f:
    json.dump(data, f, indent=2, ensure_ascii=False)
```

### 种子处理

种子值 `-1` 表示随机，但某些工作流可能不支持 `-1`。需要转换为随机正整数:

```python
if seed == -1:
    import random
    actual_seed = random.randint(0, 2**31 - 1)
else:
    actual_seed = seed
```

## 项目结构

```
ComfUI-Api-Demo/
├── main.py                 # 主程序入口
├── pyproject.toml          # 项目配置和依赖
├── requirements.txt        # 依赖列表
├── config/                 # 配置文件目录
│   ├── __init__.py        # 配置包初始化
│   ├── config.py          # 主配置文件
│   └── workflow_api.json  # 示例工作流
├── comfyui/               # ComfyUI API 工具包
│   ├── __init__.py       # 包初始化，导出主要类
│   ├── basic_sync.py     # 基础同步调用
│   ├── websocket_client.py  # WebSocket 客户端
│   ├── batch_generator.py   # 批量生成器
│   ├── image_downloader.py  # 图像下载器
│   └── comfyui_client.py   # 统一 API 客户端
└── generated_images/     # 默认图像输出目录（自动创建）
```

## 导出类

在 `comfyui/__init__.py` 中导出主要类:

```python
from .basic_sync import ComfyUISyncClient
from .websocket_client import ComfyUIWebSocketClient, ProgressMonitor
from .batch_generator import BatchGenerator
from .image_downloader import ImageDownloader
from .comfyui_client import ComfyUIUnifiedClient

__all__ = [
    "ComfyUISyncClient",
    "ComfyUIWebSocketClient",
    "ProgressMonitor",
    "BatchGenerator",
    "ImageDownloader",
    "ComfyUIUnifiedClient",
]
```

## 开发注意事项

1. **确保 ComfyUI 服务器正在运行**: 在开发前启动 ComfyUI 服务器
2. **使用正确的节点 ID**: 根据实际工作流 JSON 文件中的节点 ID 进行参数修改
3. **处理网络错误**: 添加适当的异常处理，特别是网络请求和 WebSocket 连接
4. **保持配置集中**: 将所有配置项放在 `config/config.py` 中
5. **使用 UTF-8 编码**: 读写文件时始终指定 `encoding="utf-8"`
6. **提供清晰的反馈**: 使用 `print()` 提供操作进度的清晰反馈

## 常见错误排查

### 连接失败
- 检查 ComfyUI 服务器是否运行
- 检查服务器地址和端口是否正确
- 确认防火墙设置

### 节点 ID 不存在
- 检查工作流 JSON 文件中的实际节点 ID
- 使用 `client.get_workflow()` 查看所有节点

### 提示词不生效
- 确认提示词节点的 ID 正确
- 检查节点 ID 与参数名的对应关系

---
> Source: [zer0Black/ComfyUI-Api-Demo](https://github.com/zer0Black/ComfyUI-Api-Demo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-03 -->
