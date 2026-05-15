## java-wiki

> 本项目目的是将大型java代码库解析成知识图谱并存入到neo4j中，neo4j中每个点对应代码中的一个实体，主要类型有：

# 项目介绍
本项目目的是将大型java代码库解析成知识图谱并存入到neo4j中，neo4j中每个点对应代码中的一个实体，主要类型有：
- Block：模块
- Directory：目录
- Package：包
- File：文件
- Method：方法
- Class：类
- Enum：枚举
- Interface：接口
- Record：记录
- Annotation：注解
- Field：字段
- Enumconstant：枚举常量

其中Block是基于项目目录和文件源码信息划分的模块

实体之间连边表示它们之间的依赖关系，边的类型有：
- ANNOTATES：表示注解作用于哪个实体, 包括Annotation-[:ANNOTATES]->Method, Annotation-[:ANNOTATES]->Class, Field<-[:ANNOTATES]-Annotation
- CALLS：表示方法调用, 包括Method-[:CALLS]->Method
- CONTAINS：表示Package包含哪些文件实体, 包括Package-[:CONTAINS]->File
- DECLARES：表示实体定义, 包括File-[:DECLARES]->Class, File-[:DECLARES]->Enum, File-[:DECLARES]->Annotation, File-[:DECLARES]->Interface, File-[:DECLARES]->Record, Class-[:DECLARES]->Class, Enum-[:DECLARES]->Field, Class-[:DECLARES]->Field, Enum-[:DECLARES]->Method, Class-[:DECLARES]->Method, Interface-[:DECLARES]->Method, Enum-[:DECLARES]->Enumconstant
- DIR_INCLUDE：表示目录之间、目录和文件之间的包含关系, 包括Directory-[:DIR_INCLUDE]->Directory, Directory-[:DIR_INCLUDE]->File
- EXTENDS：表示继承关系, 包括Class-[:EXTENDS]->Class
- HAS_TYPE：表示某个字段的类型与某个类、接口或枚举等实体相关, Field-[:HAS_TYPE]->Class, Field-[:HAS_TYPE]->Enum
- IMPLEMENTS：表示接口实现, 包括Class-[:IMPLEMENTS]->Interface
- RETURNS：表示方法返回, 包括Method-[:RETURNS]->Class, Method-[:RETURNS]->Interface
- USES：表示方法中使用了某个类、接口或枚举等实体，包括Method-[:USES]->Class, Method-[:USES]->Field
- f2c：表示Block之间的包含关系，包括Block-[:f2c]->Block, Block-[:f2c]->File, Block树根节点是name属性为root的Block节点

## 节点属性详细说明

### 通用属性（所有节点）
- **nodeId**：节点唯一标识id（整数类型）
- **name**：实体名称

### 代码实体节点属性（Class、Interface、Enum、Method、Field等）
除了通用属性外，还包含以下属性：
- **source_code**：实体对应的完整源码
- **background**：实体所在文件的背景信息
- **semantic_explanation**：语义解释（JSON字符串格式），包含以下字段：
  - SE_What: 对该节点的描述
  - SE_Why: 该节点的设计目的和选择意义
  - SE_When: 描述该节点的使用者，使用该节点的时机和位置
  - SE_How: 该节点是如何工作的（控制流程）
  - SE_unsure_part: 对该实体节点解释中不确定的部分
- **modifiers**：访问修饰符（如public、private、static等），多个修饰符用空格或换行分隔

### File节点属性
- **nodeId**：节点唯一标识id
- **name**：文件名（包含路径）
- **source_code**：文件完整源码
- **module_explaination**：该文件的功能说明
- **semantic_explanation**：文件的语义解释（JSON字符串格式）

### Package节点属性
- **nodeId**：节点唯一标识id
- **name**：包名（如com.example.service）
- **semantic_explanation**：包的语义解释（JSON字符串格式）

### Block节点属性
- **nodeId**：节点唯一标识id
- **name**：模块名称（根节点name为"root"）
- **module_explaination**：介绍该Block的功能和职责
- **child_blocks**：子节点列表（JSON字符串数组），每个元素格式为`{"nodeId": "xxx"}`，例如`[{"nodeId": "318"},{"nodeId": "327"}]`
- **parent_blocks**：父节点列表（JSON字符串数组），格式同child_blocks

### Directory节点属性
- **nodeId**：节点唯一标识id
- **name**：目录名称
- 注：Directory节点之间的包含关系由DIR_INCLUDE边表示，节点本身不存储额外信息

# Workflow架构要求

## 整体架构
每个workflow文件（位于gy/workflows目录下）是一个完整的工作流，用于生成一个完整的页面。workflow应该：
1. 使用langgraph框架搭建状态图
2. 并发生成页面的各个章节（使用asyncio.gather等并发机制）
3. 最终整理为统一的JSON格式输出

## 技术栈要求
1. **框架**：基于langgraph实现状态图工作流
2. **数据库接口**：使用gy/interfaces/neo4j_interface.py中的Neo4jInterface
3. **LLM接口**：使用gy/interfaces/llm_interface.py中的LLMInterface
4. **提示词管理**：工作流所需提示词写在chains/prompts目录下，一个工作流对应一个或多个提示词文件
5. **环境变量**：项目所需环境变量在.env文件中配置

## 并发性能优化

### 并发配置参数
在.env文件中配置以下参数来控制并发行为：

- **MAX_CONCURRENT_BLOCKS**：中间层Block文档生成的最大并发数（默认：10）
  - 推荐值：10-20（取决于LLM API的RPM限制）
  - GPT-4 Tier 1（500 RPM）：10-20
  - GPT-4 Tier 2（5000 RPM）：50-100

### 并发策略
1. **完全并发模式**（internal_block_workflow.py使用）：
   - 先一次性遍历整棵Block树，预先计算所有路径
   - 使用asyncio.Semaphore控制最大并发数
   - 使用asyncio.gather并发执行所有文档生成任务
   - 适合Block数量较多的情况，显著提升性能

2. **层级并发模式**（可选实现）：
   - 同一层级的Block并发处理
   - 必须等当前层级全部完成后才能处理下一层
   - 适合需要严格控制依赖关系的场景

### 性能对比
假设有100个中间层Block，每个Block需要2次LLM调用，每次调用平均10秒：
- **串行执行**：100 × 2 × 10秒 = 33分钟
- **并发执行（并发数10）**：约3-5分钟（提升6-10倍）

## 重要约束
- **不要将任何信息写入neo4j**，只能读取数据
- 所有写操作仅限于生成输出文件（JSON、Markdown等）

## 输出格式规范

### 页面JSON结构
每个workflow生成一个完整页面的JSON文件，结构如下：

```json
{
    "wiki": [
        {
            "markdown": "# 1. 概述\n## 1.1 系统架构\n## 1.2 核心功能",
            "neo4j_id": {
                "1.1": ["111", "222"],
                "1.2": ["333", "444"]
            }
        },
        {
            "mermaid": "# 2. 类关系图\n\n```mermaid\ngraph TD\n  A1[UserService] --> B1[UserDao]\n```",
            "mapping": {
                "A1": "1234",
                "B1": "5678"
            }
        },
        {
            "mermaid": "# 3. 模块依赖图\n\n```mermaid\ngraph TD\n  M1[用户模块] --> M2[订单模块]\n```",
            "neo4j_id": {
                "M1": "18437",
                "M2": "18438"
            }
        }
    ],
    "source_id_list": [
        {
            "source_id": "1234",
            "name": "com/example/service/UserService.java",
            "lines": ["10-50", "60-80"]
        },
        {
            "source_id": "5678",
            "name": "com/example/dao/UserDao.java",
            "lines": ["15-45"]
        }
    ]
}
```

### 字段详细说明

#### 1. wiki数组
- 包含页面的所有章节，每个元素代表一次LLM调用生成的章节
- 章节按顺序排列，形成完整的页面内容

#### 2. 章节类型

**Markdown章节**：
```json
{
    "markdown": "# 标题\n## 小标题1\n内容...\n## 小标题2\n内容...",
    "neo4j_id": {
        "小标题1": ["nodeId1", "nodeId2"],
        "小标题2": ["nodeId3", "nodeId4"]
    }
}
```
- `markdown`：包含完整的Markdown文本，支持多级标题
- `neo4j_id`：记录每个小标题引用的neo4j节点ID列表
  - 键：小标题文本（如"1.1"、"系统架构"等）
  - 值：nodeId数组，表示该小标题内容引用的所有节点

**Mermaid章节（有源码定位）**：
```json
{
    "mermaid": "# 图表标题\n\n```mermaid\ngraph TD\n  A1[类A] --> B1[类B]\n```",
    "mapping": {
        "A1": "1234",
        "B1": "5678"
    }
}
```
- `mermaid`：包含标题和mermaid图表代码
- `mapping`：将mermaid节点ID映射到source_id
  - 键：mermaid图中的节点ID（如"A1"、"B1"）
  - 值：source_id字符串，对应source_id_list中的条目

**Mermaid章节（无源码定位）**：
```json
{
    "mermaid": "# 模块关系图\n\n```mermaid\ngraph TD\n  M1[模块A] --> M2[模块B]\n```",
    "neo4j_id": {
        "M1": "18437",
        "M2": "18438"
    }
}
```
- 用于Block、Package等无法定位到具体源码行号的节点
- `neo4j_id`：将mermaid节点ID映射到neo4j的nodeId
  - 键：mermaid图中的节点ID
  - 值：neo4j中的nodeId字符串

#### 3. source_id_list数组
- 页面级别的源码定位信息列表
- **重要**：source_id在整个页面中必须唯一
- 每个元素包含：
  - `source_id`：唯一标识符（建议使用4位数字字符串）
  - `name`：源文件的相对路径
  - `lines`：源码行号范围数组
    - 单行：`"30"`
    - 范围：`"10-25"`
    - 多个范围：`["10-25", "30", "40-50"]`

#### 4. mapping vs neo4j_id 使用规则
- **mapping + source_id_list**：用于可以定位到具体源码的节点（Class、Method、Interface等）
- **neo4j_id**：用于无法定位源码的抽象节点（Block、Package等）
- **二选一**：同一个mermaid章节中，要么使用mapping，要么使用neo4j_id，不能同时使用

## Langgraph工作流开发最佳实践
- 所有需要在节点间传递的数据都要在State中定义
- 节点函数通过返回字典来更新state
- 被调用的workflow必须在其节点函数中返回数据到state
- 调用者通过`result`获取最终state中的数据
- 不需要通过文件系统传递数据
## 提示词设计规范

**返回JSON格式的提示词**：
```python
PROMPT = PromptTemplate(
    input_variables=["data"],
    template="""
你的任务是生成...

## 输出格式
必须返回严格的JSON格式，不要包含任何其他文字。

---
### 输入数据:
```
{data}
```
---

【输出格式】（严格JSON，不要包含任何其他解释、前言或Markdown标记）
{{"markdown": "这里是markdown内容", "mapping": {{"key": "value"}}}}
"""
)
```

**关键点**：
- 明确说明"严格JSON"、"不要包含其他文字"
- 提供JSON格式示例
- 使用双大括号`{{}}`转义JSON中的花括号

---
> Source: [gyyggyg/java_wiki](https://github.com/gyyggyg/java_wiki) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-13 -->
