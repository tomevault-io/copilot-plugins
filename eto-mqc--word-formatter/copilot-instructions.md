## word-formatter

> >


# Word 文档智能排版

基于 python-docx 的确定性排版引擎，1:1 复刻格式规范，杜绝 AI 自行发挥。

## 核心设计原则

1. **确定性规则分类** —— 用正则 + OOXML 属性判定元素类型（标题/正文/表格等），不用 AI 猜
2. **强制字体应用** —— 逐 run 设置字体，剥离 `w:eastAsiaTheme` 等 theme 属性，防止 Word 主题自动替换
3. **OOXML 级缩进** —— 先清除所有残留缩进属性（`w:hanging`/`w:leftChars` 等），再设 `firstLineChars=200`，杜绝缩进叠加
4. **安全副本** —— 不修改原文件，操作临时副本
5. **校验闭环** —— 排版后自动校验每一项，列出所有不符合项，必须人工确认

## 依赖

```bash
pip install python-docx>=1.1.0
```

## 执行流程

### Step 1 确认需求 —— 两种格式输入方式

**方式 A：从参考文档提取格式（VR-A）**

用户给一份"排版好的参考文档"，引擎自动提取其中的格式规范，然后应用到目标文档。

```
用户：我有一份排版好的 XX.docx，帮我按这个格式排版 YY.docx
  ↓
Step 1: 提取参考文档的格式
  python scripts/formatter_extract.py <参考.docx> -o /tmp/extracted.json
  ↓
Step 2: 把提取结果展示给用户确认
  "提取到的格式：标题=黑体二号居中，正文=宋体小四首行缩进..."
  ↓
Step 3: 用户确认或修改后应用
  python scripts/formatter_cli.py <目标.docx> --preset /tmp/extracted.json --verify
```

```bash
# 一步到位：提取 + 直接应用
python scripts/formatter_extract.py <参考.docx> --apply <目标.docx>

# 只提取不应用（查看格式规范）
python scripts/formatter_extract.py <参考.docx> -o preset.json
```

**方式 B：自然语言描述格式（VR-B）**

用户直接说想要什么格式，AI 解析为预设 JSON。如果描述不够明确，AI 必须追问。

```
用户：帮我排版，标题用黑体二号，正文用宋体小四，行距1.5倍
  ↓
Step 1: AI 解析用户描述，生成预设 JSON
  如果不确定（如"行距1.5倍"是固定值还是倍数？），必须问用户
  ↓
Step 2: 生成预设并展示给用户确认
  "确认格式：标题=黑体22pt居中，正文=宋体12pt首行缩进，行距=固定值20pt..."
  ↓
Step 3: 用户确认后应用
  python scripts/formatter_cli.py <目标.docx> --preset /tmp/custom.json --verify
```

**预设快捷方式（内置规范）**

不想从头描述时，可直接用内置预设：
- `gongwen` —— 党政机关公文格式（GB/T 9704-2012）
- `thesis` —— 学术论文通用格式

### Step 2 运行排版

```bash
# 公文排版（默认）
python scripts/formatter_cli.py <input.docx> --preset gongwen --verify

# 论文排版
python scripts/formatter_cli.py <input.docx> --preset thesis --verify

# 自定义预设（方式A提取 或 方式B生成 的 JSON）
python scripts/formatter_cli.py <input.docx> --preset <path.json> --verify

# 只校验不排版（检查现有文档是否符合规范）
python scripts/formatter_cli.py <input.docx> --verify-only

# 覆盖参数
python scripts/formatter_cli.py <input.docx> --set title_size=18 body_font=宋体

# 列出可用预设
python scripts/formatter_cli.py --list-presets
```

输出文件默认为 `<原文件名>_formatted.docx`，可用 `-o` 指定。

### Step 3 审查校验报告

排版后自动运行校验，输出格式为：

```
校验结果：X 个错误 / Y 个警告 / Z 个信息

❌ 错误（必须修复）：
  [段落3/run1] 字号不符
    期望: 16pt  |  实际: 12pt

⚠️ 警告（建议修复）：
  [段落5] 行距不符
    期望: 28pt  |  实际: 24pt
```

**AI 必须做的**：把校验报告原样呈现给用户，不要自行解释或忽略任何 ERROR。

### Step 4 给用户摘要

在对话中给出：
- 排版完成的文件路径
- 校验结果摘要（通过/未通过）
- 如有 ERROR，列出具体哪些段落有问题，让用户确认

## 排版规范速查

### 公文排版（gongwen）

| 元素 | 字体 | 字号 | 行距 | 对齐 | 缩进 |
|------|------|------|------|------|------|
| 标题 | 方正小标宋简体 | 二号(22pt) | 33pt | 居中 | 无 |
| 副标题 | 楷体_GB2312 | 三号(16pt) | 28pt | 居中 | 无 |
| 一级标题 | 黑体 | 三号(16pt) | 28pt | 两端对齐 | 首行2字符 |
| 二级标题 | 楷体_GB2312 | 三号(16pt) | 28pt | 两端对齐 | 首行2字符 |
| 三级标题 | 仿宋_GB2312 | 三号(16pt) | 28pt | 两端对齐 | 首行2字符 |
| 正文 | 仿宋_GB2312 | 三号(16pt) | 28pt | 两端对齐 | 首行2字符 |

页边距：上 3.7cm / 下 3.5cm / 左 2.8cm / 右 2.6cm

### 论文排版（thesis）

| 元素 | 字体 | 字号 | 行距 | 对齐 |
|------|------|------|------|------|
| 标题 | 黑体 | 二号(22pt) | 33pt | 居中 |
| 一级标题 | 黑体 | 三号(16pt) | 28pt | 居中 |
| 二级标题 | 黑体 | 四号(14pt) | 24pt | 左对齐 |
| 三级标题 | 黑体 | 小四(12pt) | 20pt | 左对齐 |
| 正文 | 宋体 | 小四(12pt) | 20pt | 两端对齐 |

页边距：上下 2.54cm / 左右 3.17cm

## 标题层级识别规则

| 层级 | 正则 | 示例 |
|------|------|------|
| H1 | `^一/二/三...、` | 一、总体要求 |
| H2 | `^（一/二/三...）` | （一）基本原则 |
| H3 | `^\d+\. ` | 1. 工作目标 |
| H4 | `^（\d+）` | （1）第一项 |

## 校验项清单

- [ ] 页面设置：页边距、纸张大小
- [ ] 标题：字体、字号、对齐、行距
- [ ] 正文：字体、字号、行距、首行缩进
- [ ] 表格：表头字体、正文字体、边框
- [ ] 页码：是否存在 PAGE 域代码
- [ ] 字体残留：是否有未清除的 theme 属性
- [ ] 缩进叠加：是否有重复的缩进属性

## 参考文档

- [格式规范详解](references/formatting-rules.md) —— 所有检测规则和默认值
- 参考实现：[Word-Formatter-Pro](https://github.com/cwyalpha/Word-Formatter-Pro)

---
> Source: [ETO-MQC/word-formatter](https://github.com/ETO-MQC/word-formatter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
