## oshw-readme-skill

> > 核心规则、模板结构、设计风格规范均在 `SKILL.md` 中。

# AGENTS.md — Supplementary Agent Guide

> 核心规则、模板结构、设计风格规范均在 `SKILL.md` 中。
> 本文件仅覆盖 `SKILL.md` 中未涉及的约定和工具使用指导，不重复内容。

## 项目上下文

**输出格式**: HTML body 内容（纯内联样式） · **运行环境**: 浏览器渲染（立创开源广场 README）

## HTML 生成规则

### 绝对禁止

| 禁止项 | 原因 |
|--------|------|
| `<style>` 标签 | 立创开源广场只支持内联样式 |
| `<script>` 标签 | 安全限制，不允许脚本执行 |
| 空行 | 输出必须是连续的 HTML，行间无空行 |
| `class` 属性 | 不支持外部样式表，class 无效 |
| 外部 CSS 引用 | 无法加载外部资源 |
| JavaScript 事件 | `onclick` 等事件属性无效 |

### 必须遵守

| 规则 | 说明 |
|------|------|
| 内联 style | 所有样式通过 `style=""` 属性设置 |
| light-dark() | 颜色值使用 `light-dark(亮色,暗色)` 函数 |
| 图片路径 | 统一使用 `img/filename.png` 相对路径 |
| 图片样式 | 每个 `<img>` 必须包含 `border-radius` 和 `margin-bottom` |
| 容器宽度 | 主内容区 `max-width:1100px` |
| 字体栈 | 外层 wrapper 使用 `-apple-system,BlinkMacSystemFont,` |

## 内联样式编写规范

### 简写属性无空格

```html
<!-- ✅ 正确 -->
<div style="display:flex;gap:24px;padding:32px">

<!-- ❌ 错误 -->
<div style="display: flex; gap: 24px; padding: 32px">
```

### 颜色值格式

```html
<!-- ✅ 正确 — 使用 light-dark() -->
color: light-dark(#0f0c29, #ffffff)
background: light-dark(#ffffff, #1e293b)

<!-- ✅ 正确 — 单一颜色（无需深色模式） -->
color: #7c3aed

<!-- ❌ 错误 — 不支持 CSS 变量 -->
color: var(--text-primary)
```

### 渐变语法

```html
<!-- ✅ 正确 -->
background: linear-gradient(135deg, #7c3aed, #a855f7)
background: linear-gradient(160deg, light-dark(#ffffff,#1e293b), light-dark(#f8fafc,#0f172a))

<!-- ❌ 错误 — 不支持 CSS 变量 -->
background: linear-gradient(135deg, var(--primary), var(--secondary))
```

## 图片匹配规范

### 匹配规则

**不要求用户按固定格式命名图片**。根据文件名中的文字描述自动匹配到对应 section：

| 目标模块 | 文件名关键词（不区分大小写） |
|----------|---------------------------|
| Hero 主图 | `hero`, `主图`, `渲染`, `产品`, `正面`, `效果图`, `展示` |
| 组成图 | `composition`, `组成`, `架构`, `框图`, `模块图`, `系统` |
| 功能介绍 | `feature`, `功能`, `特性`, `演示`, `效果`, `截图`, `蓝牙`, `WiFi`, `显示`, `遥控`, `传感` |
| PCB 设计图 | `pcb`, `PCB`, `布局`, `板图`, `设计图` |
| 原理图 | `schematic`, `原理图`, `电路图`, `电路` |
| 3D 外壳 | `3d`, `3D`, `外壳`, `壳体`, `正面`, `底壳`, `顶壳`, `爆炸`, `拆解` |
| 组装效果 | `assembly`, `组装`, `装配`, `成品`, `实物` |
| 引脚定义 | `pinout`, `引脚`, `接口`, `排针` |
| BOM 相关 | `bom`, `BOM`, `元器件`, `物料`, `清单` |

### img 标签格式

使用实际匹配到的文件名（含中文/英文），不用占位符名称：

```html
<img src="img/ESP32智能小车正面渲染图.png" style="width:100%;border-radius:12px;margin-bottom:48px" />
<img src="img/蓝牙遥控功能展示.jpg" style="width:100%;border-radius:12px;margin-bottom:48px" />
```

### 双图并排格式

```html
<div style="display:grid;grid-template-columns:1fr 1fr;gap:16px">
<img src="img/底壳3D渲染.png" style="width:100%;border-radius:12px" />
<img src="img/顶壳3D渲染.png" style="width:100%;border-radius:12px" />
</div>
```

## 文件夹创建规则

根据用户确认的项目能力，按以下规则创建文件夹：

| 条件 | 创建的文件夹 |
|------|-------------|
| 始终 | `img/` |
| 始终 | `code/` |
| 有原理图 AND 有 PCB | `SRC/` |

### SRC 文件夹说明

当用户确认项目有原理图和 PCB 时：
- 提示用户将嘉立创EDA导出的网表和 BOM 文件放入 `SRC/` 文件夹
- 网表格式为 `.enet`（嘉立创EDA专用 JSON 网表），BOM 格式为 `.csv` 或 `.xlsx`
- **自动读取**：使用 Glob 查找 `SRC/` 下的 BOM 文件（匹配 `*BOM*`, `*bom*`），使用 Read 读取内容
- **自动解析**：将 BOM CSV/XLSX 数据解析为表格，直接用于生成 BOM section
- 网表文件仅确认存在，不在页面中展示

## 交互式信息收集流程

使用 AskUserQuestion 工具收集信息时，**先完成文件扫描**，再基于扫描结果提问：

### 前置步骤（必须先执行）

1. 使用 Glob 扫描 `img/` 文件夹，列出所有图片
2. 使用 Read 读取 `code/` 中的关键源码文件
3. 使用 Read 读取 `SRC/` 中的 BOM 文件（.csv/.xlsx）和网表文件（.enet）
4. 根据图片文件名关键词匹配图片到 section

### 第一轮问题（必填，仅询问文件扫描无法获取的信息）

1. 项目名称（文本输入）
2. 项目简介/一句话描述（文本输入）
3. 项目类型（单选：开发板/传感器模块/IoT设备/工具设备/其他）

### 第二轮问题（基于扫描结果确认）

4. 项目包含哪些内容（多选：原理图/PCB/代码/3D外壳）
   - **预填已确认项**：如果 `img/` 中已匹配到原理图/PCB/3D 图片，或 `code/` 中已有源码，自动勾选对应选项

### 第三轮问题（仅询问无法自动推断的信息）

5. 主控芯片 — 如果从源码（如 `platformio.ini`、`#include` 头文件）可推断，预填后确认
6. 主要功能模块 — 如果从源码结构可推断，列出后让用户确认/修改
7. BOM 数据 — 如果 `SRC/` 中已读取到 BOM 文件，自动使用；否则询问用户

### 不再需要的信息

- ~~图片清单~~ — 已通过扫描自动获取
- ~~源码目录结构~~ — 已通过读取 `code/` 自动获取
- ~~BOM 手动输入~~ — 已通过读取 `SRC/` 自动获取

## 输出流程

### 1. 扫描和读取文件

- 使用 Glob 扫描 `img/`、`code/`、`SRC/` 文件夹
- 使用 Read 读取 `code/` 中的关键源码文件（main.cpp, config.h, platformio.ini 等）
- 使用 Read 读取 `SRC/` 中的 BOM 文件（.csv/.xlsx），解析为表格数据
- 使用 Read 读取 `SRC/` 中的网表文件（.enet），解析 JSON 数据

### 2. 匹配图片

根据 `img/` 中图片文件名的文字描述，按关键词规则自动匹配到对应 section。输出匹配结果供用户确认。

### 3. 收集信息

基于已读取的文件内容，通过 AskUserQuestion 仅询问缺失信息。已从文件获取的信息（如 BOM 数据、源码结构、项目能力）自动填入，不再重复提问。

### 4. 分段生成 HTML

**不要一次性生成整个 HTML**。按以下顺序逐段生成并写入 `index.html`：

1. 写入外层 wrapper 开始标签（`<div style="color-scheme:light;...">` + `<div style="max-width:1100px;...">`）
2. 逐个 section 生成并追加：Hero → 组成图 → 功能介绍 → 硬件详解 → 3D外壳 → BOM → 快速开始 → 源码 → 链接
3. 写入外层 wrapper 闭合标签（`</div></div>`）

每次追加时确保与已有内容之间无空行。

### 5. 输出使用报告

向用户提供图片使用报告，格式如下：

```
本次生成使用了以下资源：

[已匹配图片]
✅ ESP32智能小车正面渲染图.png → Hero 主图
✅ 项目组成架构图.png → 组成图
✅ 蓝牙遥控功能展示.png → 功能介绍

[已读取数据]
✅ BOM_项目名称.csv → 已解析 28 个元器件，生成 BOM 表格
✅ code/src/main.cpp → 已读取源码结构

[未匹配图片]
⚠️ IMG_20240101.png — 未能识别用途，请用描述性文字重命名

[缺失内容]
❌ 硬件详解 — 未找到 PCB/原理图图片
```

### 6. 提示后续操作

告知用户：
- 未匹配的图片可以重命名后重新运行 Skill
- 可以调整已生成的 section 内容
- 刷新 index.html 查看效果

## 常见错误

| 错误 | 正确做法 |
|------|----------|
| 在 style 中使用 CSS 变量 | 使用具体颜色值或 light-dark() |
| 在 style 中使用 class 选择器 | 将样式直接写在元素的 style 属性中 |
| 图片路径使用绝对路径 | 使用 `img/filename.png` 相对路径 |
| 生成完整的 HTML 文档 | 只生成 body 内容 |
| 在 HTML 中留空行 | 移除所有空行 |
| 使用 `<br>` 标签换行 | 使用 `<div>` 或 `line-height` 控制行距 |
| 一次性生成整个 HTML 写入文件 | 分段生成，逐个 section 追加写入 |
| 未读取用户文件就开始生成 | 先扫描读取 `img/`、`code/`、`SRC/`，再开始生成 |
| 使用占位符图片名（如 `hero.png`） | 使用 `img/` 下实际文件的真实文件名 |
| 手动输入 BOM 数据 | 从 `SRC/` 中读取 BOM 文件自动解析 |

## 不要修改

- `SKILL.md` 前置元数据 — Skill 元数据
- `README.md` / `README_EN.md` — 除非用户明确要求更新

---
> Source: [JasonYANG170/oshw-readme-skill](https://github.com/JasonYANG170/oshw-readme-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-06 -->
