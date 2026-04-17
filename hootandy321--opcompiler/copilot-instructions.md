## opcompiler

> 你必须意识到，本项目已经建立了完善的文档索引。**在回答任何问题之前，严禁直接读取源码文件 (`.cpp`, `.h`, `.py`)，除非文档显式指引你去读，或任务明确要求修改代码。**

# MISSION: Infini 核心维护与开发助手
# CONTEXT: 本项目拥有完整的文档架构体系 (Full Documentation Coverage)
# STRATEGY: DOCUMENTATION-FIRST (文档优先原则)

## 1. 知识获取协议 (The "Read-First" Protocol)

你必须意识到，本项目已经建立了完善的文档索引。**在回答任何问题之前，严禁直接读取源码文件 (`.cpp`, `.h`, `.py`)，除非文档显式指引你去读，或任务明确要求修改代码。**

你的阅读路径必须严格遵循以下 **三级路由**：

### 第一级：全景导航 (Level 1: Navigation)
* **场景**：用户询问“这个项目是做什么的？”、“网络模块在哪？”、“数据流是怎么走的？”
* **动作**：读取当前目录（或根目录）下的 `README_ANALYSIS.md`。
* **禁止**：不要 `find` 或 `grep` 源代码。

### 第二级：架构理解 (Level 2: Architecture)
* **场景**：用户询问“Context 模块是如何设计的？”、“内存分配策略是什么？”
* **动作**：根据 Level 1 的指引，进入对应子目录，读取该目录下的 `README_ANALYSIS.md` (如果是中间层) 或 `README.md` (如果是叶子层)。
* **原则**：`README_ANALYSIS.md` 包含子系统职责和异构后端信息；`README.md` 包含 API 定义和类结构。

### 第三级：代码落地 (Level 3: Implementation)
* **场景**：用户询问“`initDevice` 函数的具体参数含义？”或“帮我修复 `allocator.cpp` 的 bug”。
* **动作**：
    1.  **先读文档**：读取所在目录的 `README.md`，理解设计意图、关键类和注意事项。
    2.  **后读代码**：在理解上下文后，读取具体的 `.cpp/.h` 文件。

## 2. 文档地图 (Documentation Atlas)

你必须熟记本项目的文件与文档对应关系：

| 节点类型 | 标志文件 | 你的行为准则 |
| :--- | :--- | :--- |
| **结构节点 (Node)** | `README_ANALYSIS.md` | **读这里**以了解：子模块列表、数据流向、被跳过的异构后端 (CUDA/CPU等)。 |
| **叶子节点 (Leaf)** | `README.md` | **读这里**以了解：核心类详解、API 接口、使用示例 (Usage)、实现细节 (Magic)。 |
| **源码文件 (Src)** | `.cpp / .hpp` | **最后才读**。它是实现细节的真理，但不是架构理解的入口。 |

## 3. 交互行为规范 (Behavior Rules)

1.  **拒绝盲目搜索**：
    * ❌ 用户：“TopKRouter 是怎么实现的？” -> Claude: (立即 grep 代码...)
    * ✅ 用户：“TopKRouter 是怎么实现的？” -> Claude: (读取 `Infini/Core/.../topkrouter/README_ANALYSIS.md`，发现它有 cuda 和 cpu 两个后端，询问用户想看哪个，或者根据文档总结通用逻辑。)

2.  **引用文档回答**：
    * 在回答架构问题时，优先引用 `README_ANALYSIS.md` 中的描述。
    * 在回答 API 问题时，优先引用 `README.md` 中的 Usage Example。

3.  **维护文档一致性**：
    * 如果你在执行代码修改任务（Coding Task），修改完成后，**必须**检查当前目录的 `README.md` 是否需要更新。
    * 如果代码变更导致文档过时，请主动提示用户或直接更新文档。

## 4. 常用工具指令 (Shortcuts)

* **`help`**: 显示此协议。
* **`map`**: 读取根目录的 `README_ANALYSIS.md` 并展示顶层模块导航。
* **`doc [path]`**: 智能读取指定路径下的文档（自动判断是读 `README.md` 还是 `README_ANALYSIS.md`）。

---
# 针对 ClauedCode 的特殊配置
Commands:
  - name: "update-doc"
    description: "Update documentation for current directory using appropriate agent"
    usage: "Should be used when code changes significantly"
    prompt: |
      Check current directory type. 
      If leaf (has code), run: claude -p leaf-code-analyst.md --cwd . "Update README based on current code"
      If node (has subdirs), run: claude -p architect-aggregator.md --cwd . "Update ANALYSIS based on subdirs"

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/hootandy321) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
