## extractdoc

> - **适用对象**：`d:/Python-Learning/extract_doc/` 工作区内的全部文件与对话。



# 项目工作空间规则

## 适用范围
- **适用对象**：`d:/Python-Learning/extract_doc/` 工作区内的全部文件与对话。
- **语言要求**：所有与 Cascade 的交流、代码注释与文档内容需使用中文。

## 核心原则

### 1. 禁止伪造或生成假数据
- 数据缺失时必须如实为空或抛出异常，不得补全示例或占位数据。
- **良好示例**：
  ```python
  if not flight_data:
      raise ValueError("未能提取到航班数据")
  ```
- **不良示例**：
  ```python
  if not flight_data:
      flight_data = [{"airline": "未知航空公司", ...}]
  ```

### 2. 回复与内容真实性
- 所有回答必须使用中文并基于事实，不得凭空猜测或捏造结论。
- **良好示例**：根据代码分析错误原因并给出可验证的修改建议。
- **不良示例**：笼统建议"重启服务器"而无事实依据。

### 3. 方案需附出处
- 提供编程方案时需注明参考文档或标准规范，严禁虚构引用。
- **良好示例**：引用官方文档链接并给出对应代码片段。

## 代码规范

### 注释与文档字符串
- 代码注释、文档字符串统一使用中文，解释功能、参数与返回值。
- Python 文件应在开头包含 UTF-8 编码声明与文件说明块。
- 示例：
  ```python
  # -*- coding: utf-8 -*-
  """
  模块说明
  
  详细描述模块功能
  """
  ```

### 导入与命名约定
- 模块导入顺序：标准库 → 第三方库 → 项目模块。
- 变量与函数使用下划线命名法；类使用驼峰命名法。
- 示例：
  ```python
  # 标准库
  import os
  import json
  
  # 第三方库
  import requests
  from openai import OpenAI
  
  # 项目模块
  from utils.config_manager import config
  ```

### 类型提示
- 添加必要的类型提示以提升可读性与性能。
- 函数签名添加类型提示，优先使用 Pydantic 模型。
- 示例：
  ```python
  def extract_content(url: str) -> Dict[str, Any]:
      """提取内容"""
      pass
  ```

### 文件头部风格一致性
- 保持源文件头部注释格式一致，包含文件信息与描述。

## 配置管理（重要）

### 配置访问规则
- **必须**通过 `utils.config_manager` 访问配置。
- **严禁**硬编码任何配置参数到代码中。
- **严禁**通过命令行参数传入配置。
- 配置缺失时须抛出异常，不得在代码中设置默认值。
- 所有配置从 `config/application.yaml` 读取。

### 绝对禁止hardcode的参数
包括但不限于：
- **AI参数**：`temperature`、`max_tokens`、`model`、`api_key`
- **网络参数**：`timeout`、`retry_count`、`base_url`
- **文件路径**：输入/输出目录、模板路径、日志路径
- **业务参数**：批次大小、阈值、限制数量、页数限制

### 良好示例
```python
# ✅ 正确：从配置读取
from utils.config_manager import config

class MyService:
    def __init__(self):
        self.temperature = config.get("ai_document_analysis.temperature")
        self.max_tokens = config.get("ai_document_analysis.max_tokens")
        self.model = config.get("ai_document_analysis.model")
```

### 不良示例
```python
# ❌ 错误：hardcode参数
class MyService:
    def __init__(self):
        self.temperature = 0.01  # 禁止！
        self.max_tokens = 8192   # 禁止！
        self.timeout = 30        # 禁止！
```

### 违反后果
- 导致配置管理混乱
- 无法统一调整参数
- 违反项目架构原则
- 代码审查不通过

## 日志与异常处理

### 日志规范
- 使用 `utils.logger.setup_logger()` 初始化日志记录器。
- 捕获异常后记录详细错误信息，**禁止使用 `print` 输出**。
- 所有日志写入项目根目录下的 `logs/`。
- 示例：
  ```python
  import logging
  logger = logging.getLogger(__name__)
  
  try:
      result = process_data()
  except Exception as e:
      logger.error(f"处理数据失败: {e}", exc_info=True)
      raise
  ```

### 异常处理
- 在函数开头处理异常与边界情况。
- 不要吞掉异常，必须记录或重新抛出。
- 提供有意义的错误信息。

## 工具与依赖

### 复用工具类
- 优先使用 `utils/` 目录下现有工具类，不重复造轮子。
- 现有工具类包括：
  - `utils.config_manager`：配置管理
  - `utils.logger`：日志管理
  - `utils.translation`：翻译工具

### Poetry 依赖管理
- 使用 Poetry 管理依赖与虚拟环境。
- **禁止**使用 `pip` 直接安装。
- **禁止**在全局环境安装依赖。

### 虚拟环境隔离
- 每个项目使用独立虚拟环境。
- 禁止直接在系统环境安装依赖。

## 框架规范

### FastAPI 规范
- 纯函数使用 `def`，异步操作使用 `async def`。
- 路由定义需声明返回类型。
- 使用依赖注入管理共享资源。
- 生命周期事件优先采用 `lifespan` 上下文管理器。
- 对外部 I/O 调用使用异步实现。

### Django/MVT 规范
- 遵循 Django MVT 架构。
- 优化 ORM 查询时使用 `select_related`/`prefetch_related`。

### Python 3.12 规范
- 采用 Python 3.12 推荐特性与最佳实践。
- 使用结构化模式匹配等新特性。

## 开发规范

### 聚焦功能修复
- Bug 修复与重构需聚焦指定功能，避免大范围无关修改。
- 一次只解决一个问题。

### 文档要求
- 各一级子目录需维护 `README.md`，说明子目录内文件用途。
- 重要功能需添加使用说明。

### 控制台输出字符集
- 控制台输出仅使用 ASCII 字符，避免 GBK 编码问题。
- 日志文件使用 UTF-8 编码。

## 项目特定规范

### AI内容提取
- 使用 `WebContentExtractor` 提取网页内容。
- 提取结果必须包含 `[图片]` 标记和缩进格式。
- 不要在AI分析后再次简化或修改提取的内容。

### PPT生成
- 直接使用提取的 `section['content']`，不要截断或简化。
- 保留 `[图片]` 标记用于图片插入。
- 保留缩进格式（`"  - "` 开头）用于层次结构。

### 数据结构
- 章节数据结构：
  ```python
  {
      "title": "章节标题",
      "content": [
          "要点1",
          "  - 子要点1",  # 缩进表示层次
          "[图片]图片说明",  # 图片标记
      ],
      "level": 2
  }
  ```

## 扩展规则
- 若需扩展规则，请在 `.windsurf/rules/` 目录新增 Markdown 文件并注明适用范围。
- 新规则需与现有规则保持一致。

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/tangdazhu) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
