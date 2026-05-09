## agentic-design-patterns

> 本文档为运行在本代码库的AI编码/翻译智能体提供操作规范。

# Agentic Design Patterns - 智能体操作指南

本文档为运行在本代码库的AI编码/翻译智能体提供操作规范。

---

## 1. 项目概述

本项目是《Agentic Design Patterns》一书的中文翻译项目，是AI Agent系统设计模式的权威指南。

- **代码仓库**: https://github.com/xindoo/agentic-design-patterns
- **在线站点**: https://adp.xindoo.xyz/
- **技术栈**: Jekyll (GitHub Pages)、Markdown、Ruby
- **核心任务**: 翻译质量优化、内容更新、站点维护

---

## 2. 构建/检查/运行命令

### 环境要求

- 已安装 Ruby 2.7+ 和 Bundler
- 已安装 Node.js (用于代码检查工具)

### 依赖安装

```bash
# 安装Ruby依赖
bundle install

# 安装Node.js代码检查工具（可选，用于质量检查）
npm install -g markdownlint-cli markdown-link-check
```

### 构建与本地运行

```bash
# 构建静态站点到 _site/ 目录
bundle exec jekyll build

# 本地启动服务，支持热重载，访问地址：http://localhost:4000
bundle exec jekyll serve --livereload

# 生产环境构建（与GitHub Pages部署逻辑一致）
JEKYLL_ENV=production bundle exec jekyll build
```

### 代码检查与质量校验

```bash
# 检查所有Markdown文件的格式问题
npx markdownlint-cli chapters/

# 检查指定章节的无效链接
npx markdown-link-check chapters/[章节文件名].md

# 检查特定术语的翻译一致性
grep -r "术语原文" chapters/

# 验证Jekyll配置有效性
bundle exec jekyll doctor
```

### 测试要求

本内容类项目无自动化单元测试，所有修改必须通过：

1. Markdown语法检查无错误
2. 本地构建成功无Jekyll错误
3. 链接检查无无效内部链接
4. 翻译内容人工审核通过

---

## 3. 代码与内容风格规范

### Markdown格式规范

- **标题层级**: 严格保留原文的标题层级结构（# 一级标题、## 二级标题、### 三级标题，以此类推）
- **换行规则**: 段落之间使用1个空行分隔，行尾无多余空格
- **列表格式**: 嵌套列表保留原有缩进，无序列表使用 `-`，有序列表使用 `1.`
- **代码块**: 反引号后必须指定语言标识：
  ```python
  # 示例：带语言标识的代码块
  def example():
      return "value"
  ```
- **行内代码**: 变量名、函数名、命令行输入使用单反引号包裹
- **表格格式**: 保持Markdown表格对齐，表头翻译为中文，内容对齐方式与原文一致

### 翻译标准

#### 准确性优先

- 完整保留所有技术含义，无明确理由不得遗漏或新增内容
- 对技术术语翻译不确定时，添加译者注：`[译者注: 此处XXX术语翻译存疑，原文为XXX]`
- 技术概念使用行业标准译法，避免字面直译

#### 术语一致性

- **强制要求**: 所有术语翻译必须同时参考 `glossary.md` 和 `Index of Terms.md` 中的统一译法
- 新术语首次出现时需标注英文原文：`中文译名(English Original)`
- 后续出现统一使用中文译名即可
- 禁止为已有标准译法的技术术语自创翻译
- **重要原则**: 不要轻易批量替换关键词，必须根据上下文判断是否修改。例如："代理"可能是技术术语 proxy，"流水线"可能是计算机科学通用术语，"提示链"在某些语境下可能不需要改为"提示词链"。在进行任何替换前，必须先读取文件查看具体上下文。
- **章节固定标题译法**:
  | 原文 | 推荐译文 |  
  |---|---|  
  | At a Glance | 速览 |  
  | What | 问题背景 |  
  | Why | 解决方案 |  
  | Rule of Thumb | 实践建议 |

#### 语言风格

- 使用自然流畅的中文表达，避免直译英文句式
- 保持技术文档的专业、简洁语气
- 全书翻译风格统一，与已翻译章节保持一致
- 标点符号：中文内容使用中文标点，代码和英文术语使用英文标点

### 命名规范

- **文件名**: 章节文件名保留原文英文：`Chapter X_ [Original Name].md`
- **图片路径**: 使用 `![中文描述](../images/chapter-XX/[original-image-filename].png)` 格式
- **分支名**: 新贡献使用 `feature/[任务描述]` 格式，例如 `feature/translate-chapter-5`、`feature/fix-chapter-3-typos`
- **提交信息**: 使用清晰的中文描述：`完成第X章翻译`、`修复第Y章术语错误`、`更新Z章节内容`

### 错误处理

- 遇到无效链接时，修复为正确的内部路径
- 遇到缺失图片时，通知维护者，不得直接删除图片引用
- 发现翻译不一致时，对照 `glossary.md` 修正所有相关出现位置
- 禁止忽略Jekyll构建错误，提交前必须修复根本原因

### 代码与技术内容规则

- **代码片段**: 所有代码保留原文英文，不得翻译代码逻辑、变量名、函数名
- **代码注释**: 可以将代码注释翻译为中文，但代码本身保持不变
- **命令行说明**: 命令本身保留英文，说明部分翻译为中文
- **框架/技术名称**: LangChain、OpenAI、MCP、RAG等专有名词保留原文，除非有广泛接受的标准中文译法

---

## 4. 工作流规范

### 修改前检查

1. 操作前先阅读 `CONTRIBUTING.md` 和 `translation-guide.md`
2. 确认你要修改的内容没有已存在的开放拉取请求
3. 查看 `progress.md` 了解当前翻译进度

### 提交要求

- 所有修改聚焦于单个章节或任务，避免在一个PR中包含不相关的改动
- 完整填写PR模板，说明修改内容和原因
- PR提交前必须通过本地构建和代码检查
- PR描述中包含验证步骤，例如："本地构建成功，markdownlint检查通过，第3章内容已审核"

### 禁止操作

- 不得修改 `original/` 目录下的文件（原文备份目录）
- 未经维护者明确批准，不得修改 `_config.yml` 中的Jekyll配置
- 未经过讨论不得新增依赖或插件
- 已正确翻译的内容不得随意重写，除非修复明确错误

---

## 5. 双语版本指南

### 5.1 双语版本概述

双语版本将英文原文和中文翻译交替显示，每段英文后紧跟对应的中文翻译，代码块作为整体不分割。这种格式便于读者对照阅读，提高翻译质量和学习效率。

**访问路径**：

- 中文版: `/chapters/`
- 英文版: `/original/`
- 双语版: `/bilingual/`

### 5.2 双语文件结构

```
bilingual/
├── Chapter 1_ Prompt Chaining.md
├── Chapter 2_ Routing.md
├── ...
├── Glossary.md
├── Index of Terms.md
└── index.md
```

**文件名**：与 `chapters/` 和 `original/` 目录保持一致，使用原文英文文件名，例如 `Chapter 1_ Prompt Chaining.md`。

**文件完整性**：`bilingual/` 目录必须包含与 `chapters/` 和 `original/` 相同的完整文件集合，包括所有章节、附录、术语表和索引文件。

### 5.3 双语内容格式规范

#### 基础格式规则

双语文件必须严格按照以下格式组织内容：

```markdown
# English Title

# 中文标题

## English Section Heading

## 中文章节标题

English paragraph here.

中文翻译段落。

Another English paragraph.

另一段中文翻译。

```python
# 代码块保持完整，只出现一次，放在对应英文段落之后
code_here()
```
```

#### 标题层级规范

- **一级标题**：英文标题后紧跟中文标题，各占一行
  ```markdown
  # Chapter 1: Prompt Chaining

  # 第 1 章：提示词链
  ```

- **二级及以下标题**：英文标题后紧跟中文标题，各占一行
  ```markdown
  ## Prompt Chaining Pattern Overview

  ## 提示词链模式概述
  ```

- **标题层级一致性**：严格保持与原文相同的标题层级结构，不得随意更改

#### 段落交替规范

- **逐段对照**：每个英文段落后紧跟对应的中文翻译段落
- **空行分隔**：英文段落与中文段落之间必须有一个空行
- **段落完整性**：保持原文段落结构，不要合并或拆分段落
- **列表处理**：完整的英文列表后紧跟完整的中文列表，保持列表格式一致

  示例：
  ```markdown
  * English list item 1
  * English list item 2

  * 中文列表项 1
  * 中文列表项 2
  ```

#### 代码块处理规范

- **代码块只出现一次**：代码块紧跟在对应的英文说明段落之后，中文翻译段落之前
- **代码保持原样**：代码内容、注释、格式完全保留原文
- **语言标识保留**：保留代码块的语言标识（如 `python`、`bash` 等）

  示例：
  ```markdown
  Here's a code example:

  ```python
  def example():
      return "value"
  ```

  这是一个代码示例：
  ```

#### 特殊元素处理

- **图片引用**：图片只出现一次，放在对应的英文段落之后，中文翻译之前
- **表格**：完整的英文表格后紧跟完整的中文表格，表头和内容都要翻译
- **链接**：链接只出现一次，保持原文链接地址，链接文本可以翻译或保留英文
- **引用块**：完整的英文引用块后紧跟完整的中文引用块

### 5.4 网页版双语支持

- 导航栏自动根据当前语言显示对应目录
- 语言切换按钮支持三态切换：**中文 → 英文 → 双语 → 中文**
- 上一章/下一章导航在三种版本间正常工作
- 使用 `_layouts/bilingual.html` 布局模板渲染双语页面
- 使用 `_includes/navigation-bilingual.html` 提供双语版导航

### 5.5 PDF/EPUB 生成

GitHub Workflow (`.github/workflows/generate-pdf.yml`) 会自动生成三种版本的 PDF 和 EPUB：

| 文件 | 说明 |
|------|------|
| `agentic-design-patterns-chinese.pdf` | 中文版 PDF |
| `agentic-design-patterns-chinese.epub` | 中文版 EPUB |
| `agentic-design-patterns-en.pdf` | 英文版 PDF |
| `agentic-design-patterns-en.epub` | 英文版 EPUB |
| `agentic-design-patterns-bilingual.pdf` | 双语版 PDF |
| `agentic-design-patterns-bilingual.epub` | 双语版 EPUB |

**触发条件**：push 到 main 分支，且修改了 `chapters/**`、`original/**` 或 `bilingual/**`

### 5.6 更新双语版本的流程

#### 完整更新流程

1. **更新中文翻译** → 修改 `chapters/` 目录下的对应文件
2. **更新英文原文**（如需要）→ 修改 `original/` 目录下的对应文件
3. **同步更新双语文件** → 修改 `bilingual/` 目录下的对应文件，确保格式符合 5.3 节规范
4. **提交更改** → 包含 `chapters/`、`original/`（如修改）和 `bilingual/` 的改动
5. **自动触发 Workflow** → 生成新的 PDF/EPUB

#### 双语文件同步检查清单

在提交双语文件更新前，请确认：

- [ ] 所有标题都有对应的中英文对照
- [ ] 所有段落都按英文-中文顺序交替排列
- [ ] 代码块只出现一次且位置正确
- [ ] 图片、表格、链接等特殊元素处理正确
- [ ] 文件名与 `chapters/` 和 `original/` 保持一致
- [ ] 文件完整性检查：没有遗漏任何章节或附录
- [ ] Markdown 语法检查通过
- [ ] 本地 Jekyll 构建成功

### 5.7 双语文件生成工具（如适用）

如果项目提供了自动生成双语文件的工具，请参考工具文档使用。在没有自动化工具的情况下，必须手动按照上述规范创建和维护双语文件。

---

## 7. 双语版本维护规范

### 7.1 双语文件质量标准

- **格式一致性**：严格遵守 5.3 节的格式规范
- **内容完整性**：双语文件必须包含原文的所有内容，不得遗漏
- **翻译准确性**：中文翻译必须与 `chapters/` 目录中的翻译保持一致
- **同步及时性**：`chapters/` 或 `original/` 更新后，`bilingual/` 必须同步更新
- **无冗余内容**：代码块、图片等元素只出现一次，避免重复

### 7.2 禁止操作

- 不得直接修改 `bilingual/` 目录而不同步更新 `chapters/` 目录
- 不得在双语文件中随意调整段落顺序或合并/拆分段落
- 不得删除或重复代码块、图片等特殊元素
- 不得改变原文的标题层级结构
- 不得在未完成同步的情况下提交部分更新

---

## 8. 核心资源

- `glossary.md`: 官方术语翻译对照表
- `translation-guide.md`: 详细翻译标准与最佳实践
- `CONTRIBUTING.md`: 完整贡献流程与要求
- `progress.md`: 当前翻译进度与章节状态
- `PROJECT_STRUCTURE.md`: 代码库文件结构说明

---
> Source: [xindoo/agentic-design-patterns](https://github.com/xindoo/agentic-design-patterns) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-03 -->
