## snappaste

> This repo uses OpenSkills to manage agent skills.

# AGENTS

This repo uses OpenSkills to manage agent skills.

- This repo is an Obsidian-style knowledge base; see [Workflow/CORE.md](Workflow/CORE.md).
- Installed skills live under `.claude/skills/`
- Sync the generated table in this file with: `openskills sync`
- Avoid manually editing content between `<!-- SKILLS_TABLE_START -->` and `<!-- SKILLS_TABLE_END -->`
- For repeatable workflows, use the `openskills-manager` skill

## Global Rules (全局规则)

<global_rules priority="0">

### Local Knowledge First (本地知识优先)

在执行任务前，**必须**先检查 `<available_skills>` 中是否有匹配的 skill：
1. 若有相关 skill，先执行 `openskills read <skill-name>` 加载其内容
2. **优先遵循 skill 中记录的方法和约定**，其次才是使用模型自身的通用知识
3. 若 skill 内容与模型知识冲突，以 skill 为准（除非 skill 明显过时或错误，此时应提醒用户更新）

**原因**：skill 是用户沉淀的领域经验和个人偏好，代表了"我们这里的做法"，比通用知识更贴合实际需求。

</global_rules>

<project_constraints priority="0">

## Project Constraints (项目开发限制)

本项目是 **SnapPaste** —— 手机拍照、电脑粘贴的轻量工具。开发时必须严格遵循以下限制：

### 核心原则

| 原则 | 要求 | 禁止 |
|------|------|------|
| **快** | 局域网直传，亚秒级延迟 | 任何增加传输延迟的设计 |
| **干净** | 图片不落盘，直接进剪贴板 | 创建临时文件、缓存目录 |
| **低启动成本** | 扫码即连接 | 要求 USB 调试、安装驱动、复杂配置 |
| **心流不断** | 拍完即传，无需二次确认 | 弹窗确认、手动下载步骤 |

### 技术选型约束

**电脑端**：
- ✅ Python (Flask/FastAPI) 或 Node.js
- ✅ 自动检测局域网 IP
- ✅ 终端或弹窗显示二维码
- ✅ 系统 API 写入剪贴板（Windows: `win32clipboard` / PowerShell）
- ❌ 禁止引入重量级框架（如 Electron）

**手机端**：
- ✅ 纯静态 HTML + JS（由电脑端 serve）
- ✅ `navigator.mediaDevices.getUserMedia()` 调用摄像头
- ✅ `fetch()` POST 图片到电脑端
- ✅ PWA 支持（可添加到主屏幕）
- ❌ 禁止依赖原生 App 或需要安装的客户端

### 架构限制

```
手机端 (PWA) ──HTTP POST──▶ 电脑端 (本地服务) ──▶ 系统剪贴板
```

- 通信方式：**仅限局域网 HTTP**
- 数据格式：图片 Base64 或 Blob
- 连接方式：**二维码扫码**（含 `http://<局域网IP>:<端口>`）

### MVP 边界 (v0.1 Must Have)

开发时优先保证以下功能，其他功能属于 "Nice to Have"：

1. 电脑端：启动脚本，显示二维码
2. 电脑端：接收图片，写入 Windows 剪贴板
3. 手机端：扫码打开网页
4. 手机端：点击按钮拍照
5. 手机端：拍完自动发送（无需二次确认）

### 禁止路径 (Not-to-do)

以下方案已明确排除，**不得重新引入**：

- ❌ scrcpy 投屏（启动成本高）
- ❌ 微信/聊天工具传图（留垃圾记录）
- ❌ OBS 方案（太重，且是视频流）
- ❌ 虚拟摄像头驱动（开发复杂度过高）
- ❌ 任何需要"二次点击确认"的流程

### 核心洞察

> **问题本质是剪贴板同步，不是视频流传输。**
> 
> 你要的不是"手机摄像头共享给电脑"，而是"拍完立刻能粘贴"。

</project_constraints>

<skills_system priority="1">

## Available Skills

<!-- SKILLS_TABLE_START -->
<usage>
When users ask you to perform tasks, check if any of the available skills below can help complete the task more effectively. Skills provide specialized capabilities and domain knowledge.

How to use skills:
- Invoke: Bash("openskills read <skill-name>")
- The skill content will load with detailed instructions on how to complete the task
- Base directory provided in output for resolving bundled resources (references/, scripts/, assets/)

Usage notes:
- Only use skills listed in <available_skills> below
- Do not invoke a skill that is already loaded in your context
- Each skill invocation is stateless
</usage>

<available_skills>

<skill>
<name>docx</name>
<description>"Comprehensive document creation, editing, and analysis with support for tracked changes, comments, formatting preservation, and text extraction. When Claude needs to work with professional documents (.docx files) for: (1) Creating new documents, (2) Modifying or editing content, (3) Working with tracked changes, (4) Adding comments, or any other document tasks"</description>
<location>project</location>
</skill>

<skill>
<name>obsidian-co-curator</name>
<description>Obsidian Knowledge Base Co-Curator and Value Catalyst Agent. Use this whenever you are asked to manage, reorganize, or update this Obsidian vault (including changes to Workflow/ rules, file moves, tagging, linking, and refactors).</description>
<location>project</location>
</skill>

<skill>
<name>openskills-manager</name>
<description>使用 OpenSkills 管理本仓库的 skills：安装/更新/移除 skill，并用 openskills sync 同步更新 AGENTS.md 的 <available_skills> 列表。</description>
<location>project</location>
</skill>

<skill>
<name>pdf</name>
<description>Comprehensive PDF manipulation toolkit for extracting text and tables, creating new PDFs, merging/splitting documents, and handling forms. When Claude needs to fill in a PDF form or programmatically process, generate, or analyze PDF documents at scale.</description>
<location>project</location>
</skill>

<skill>
<name>pptx</name>
<description>"Presentation creation, editing, and analysis. When Claude needs to work with presentations (.pptx files) for: (1) Creating new presentations, (2) Modifying or editing content, (3) Working with layouts, (4) Adding comments or speaker notes, or any other presentation tasks"</description>
<location>project</location>
</skill>

<skill>
<name>prompt-iteration</name>
<description>Systematic process to design, test, and refine prompts (Treating Prompts as Code).</description>
<location>project</location>
</skill>

<skill>
<name>skill-creator</name>
<description>Guide for creating effective skills. This skill should be used when users want to create a new skill (or update an existing skill) that extends Claude's capabilities with specialized knowledge, workflows, or tool integrations.</description>
<location>project</location>
</skill>

<skill>
<name>university-exam-prep</name>
<description>大学备考苏格拉底式学习助手，专为应对"面向课本和PPT考试"设计。当用户说"帮我复习"、"准备考试"、"备考"、"期末复习"、"模拟学习"、"突击复习"等时触发。核心特点：(1)强制要求用户上传课本/PPT/考纲等原始材料——没有材料就无法有效辅导；(2)使用苏格拉底式提问法，不直接灌输而是引导思考；(3)专注于应用型和理解型内容，而非纯概念记忆；(4)模拟真实考试场景进行针对性练习；(5)分析材料识别高频考点和重难点。适用于大学各科目期中期末考试备考、突击复习、考前模拟。</description>
<location>project</location>
</skill>

<skill>
<name>xlsx</name>
<description>"Comprehensive spreadsheet creation, editing, and analysis with support for formulas, formatting, data analysis, and visualization. When Claude needs to work with spreadsheets (.xlsx, .xlsm, .csv, .tsv, etc) for: (1) Creating new spreadsheets with formulas and formatting, (2) Reading or analyzing data, (3) Modify existing spreadsheets while preserving formulas, (4) Data analysis and visualization in spreadsheets, or (5) Recalculating formulas"</description>
<location>project</location>
</skill>

</available_skills>
<!-- SKILLS_TABLE_END -->

</skills_system>

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/spdc-elm)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/spdc-elm)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
