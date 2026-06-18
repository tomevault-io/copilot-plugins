## write-wechat-article

> Use when the user asks to "写一篇公众号文章", "创作公众号内容", "写公众号文章", "生成微信公众号文章", "优化这篇文章", "润色文章", "给文章起标题", or mentions WeChat public account (微信公众号), WeChat article formatting, WeChat-compatible HTML, article cover generation, or wants to create content for WeChat public account. Make sure to use this skill whenever the user mentions WeChat, 公众号, article writing, content creation, or needs help with article formatting or title generation.


# 微信公众号文章写作技能

## 快速参考

### 触发条件
- "写一篇公众号文章" / "创作公众号内容"
- "优化这篇文章" / "润色文章"
- "生成标题" / "给文章起标题"
- 提及微信公众号、WeChat article

### 输出产物

| 文件 | 说明 |
|-----|------|
| [slug].md | Markdown 源文件 |
| [slug]-wechat.html | 微信公众号 HTML |
| cover-[slug].html | 封面图 HTML |

### 核心流程概览

```
1. 调研 → 2. 创作 → 3. 生成文案 → 4. 设计配图 → 5. 检验 → 6. 发布
```
---

## 功能

1. **创作模式** - 从头开始生成完整文章
2. **修改模式** - 优化现有文章的某个方面

---

## 技能模式

### 创作模式

当用户说"写一篇关于XXX的公众号文章"时：

**Step 1: 深度调研**
- 使用 deep-research / agent-reach / browser-use skills
- 详见：[PROCESS_GUIDE.md#调研流程](specs/PROCESS_GUIDE.md#二调研流程)

**Step 2: 文案创作**
- 交互式问答确定方向和内容
- 详见：[PROCESS_GUIDE.md#创作流程](specs/PROCESS_GUIDE.md#三创作流程)

**Step 3: 生成文档**
- 生成 Markdown 文件
- 生成微信公众号 HTML
- 详见：[FORMAT_SPEC.md](specs/FORMAT_SPEC.md)

**Step 4: 设计配图**
- **4.1 生成封面图**：生成 cover-[slug].html（左侧矩形1175×500px + 右侧方形500×500px）
- **4.2 风格匹配**：根据文章内容推荐插图风格
  - AI 分析全文内容和关键词
  - 推荐 1-3 种风格供用户选择
  - 用户确认后应用对应色值方案
  - 详见：[STYLE_PALETTE.md](specs/STYLE_PALETTE.md)
- **4.3 生成插图**：使用确认的风格生成 3-5 张插图
  - 插图类型：comparison（对比图）、ability（能力图）、process（流程图）
  - 风格主题：应用用户确认的色值方案
  - 插图类型决定结构，风格主题提供视觉样式
- 详见：[FORMAT_SPEC.md#配图设计](specs/FORMAT_SPEC.md#五配图设计规范)

**Step 5: 质量检验**
- 错别字、标点、格式检查
- 详见：[PROCESS_GUIDE.md#质量检验](specs/PROCESS_GUIDE.md#五质量检验流程)

**Step 6: 截图预览和发布**
- 使用 agent-browser 截图预览
- 发送到飞书，不要发送图片路径，要用飞书
- 详见：[PROCESS_GUIDE.md#发布流程](specs/PROCESS_GUIDE.md##六、发布流程)

### 修改模式

当用户说"优化这篇文章"或"修改标题"时：

1. **解析意图** - 识别用户想修改什么（内容/标题/排版）
2. **读取文件** - 读取目标文件
3. **执行修改** - 使用 Agent 的能力进行修改
4. **保存文件** - 保存修改后的内容
5. **返回路径** - 返回更新后的文件路径

---

## 意图识别

| 用户输入 | 意图 | 操作 |
|---------|------|------|
| "写一篇关于AI的文章" | 创作 | 生成新文章 |
| "优化这篇文章" | 润色 | 优化现有文章 |
| "调整排版" | 格式化 | 转换为微信 HTML |
| "生成标题" | 标题 | 生成标题选项 |

---

## 关键规范速查

### 微信兼容性 ✅

- 使用 `<section>` 而非 `<div>`
- 使用 `<p>` 而非 `<ul>`/`<li>`
- 表格背景色应用到 `<th>` 标签
- 禁止 `<style>` 标签

详见：[FORMAT_SPEC.md#HTML兼容性](specs/FORMAT_SPEC.md#二html-兼容性规范重要)

### 主题色

- 微信文章色：`rgb(0, 112, 192)` 或 `#0070c0`

详见：[FORMAT_SPEC.md#主题色](specs/FORMAT_SPEC.md#一主题色规范)

### 插图风格

- 14 种可用风格，详见 [STYLE_PALETTE.md](specs/STYLE_PALETTE.md)
- 风格匹配：AI 根据文章内容自动推荐合适风格
- 插图类型：comparison（对比）、ability（能力）、process（流程）

详见：[STYLE_PALETTE.md](specs/STYLE_PALETTE.md#插图风格)

### 配图规范速查

**插图数量：**
- 短文章（<1500字）：1-2张
- 中等文章（1500-3000字）：3-4张
- 长文章（>3000字）：4-5张

**插图类型：**
- comparison：对比图（左右对比 + VS 徽章）
- ability：核心能力图（深色背景 + 三栏卡片）
- process：流程图（步骤可视化 + 代码块）

**封面图布局：**
- 左侧矩形（1175×500px）+ 右侧方形（500×500px）
- 根据文章内容选择封面图风格
- 定义卡片式布局、玻璃拟态效果、大数字冲击等设计模式
- 创建封面图、对比图、核心能力图模板

详见：[FORMAT_SPEC.md#配图设计](specs/FORMAT_SPEC.md#五配图设计规范)

**插图和封面图风格（14种可选）**：
- 信息图蓝图风、商务深蓝风、暗色科技风、杂志编辑风
- 实验室蓝图风、杂志现代风、孟菲斯波普风、报纸新闻风
- 北欧极简风、复古未来主义、深蓝卡片科技风、红蓝对话风
- 蓝绿流程风、蓝色步骤风

详见：[STYLE_PALETTE.md](specs/STYLE_PALETTE.md)

---

## 风格匹配执行逻辑

当执行 **Step 4.2: 风格匹配** 时，按以下流程操作：

### 4.2.1 分析文章内容

1. 读取文章标题和正文内容
2. 合并全文进行关键词提取
3. 统计关键词出现频率

### 4.2.2 计算风格得分

对 14 种风格分别计算匹配得分：

```
风格得分 = Σ(关键词频率 × 关键词权重)
```

例如：文章出现"代码"5次、"开发"3次、"教程"2次
- 暗色科技风得分 = 5×3 + 3×3 + 2×2 = 28
- 蓝色步骤风得分 = 5×3 + 2×2 = 19
- 信息图蓝图风得分 = 3×1 = 3

### 4.2.3 生成推荐报告

按得分降序排列，取前 3 种风格（得分 > 0），生成报告：

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
📊 文章插图风格分析报告
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

📝 文章信息
  标题：[文章标题]
  字数：约 [字数] 字
  插图建议：[数量] 张

🎯 推荐风格

  1. 【风格名称】模板文件 ⭐ 最佳匹配
     匹配得分：[得分]
     触发关键词：
       • 关键词1 (次数)
       • 关键词2 (次数)
     推荐理由：[根据适用场景生成]

     色值方案：
       背景：#[色值]
       主色：#[色值]
       次色：#[色值]
       ...

  2. 【风格名称】模板文件
     匹配得分：[得分]
     ...

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

请选择风格（输入编号或名称，或直接确认推荐风格）：
```

### 4.2.4 用户确认

1. 展示推荐报告
2. 等待用户选择（输入编号、名称，或确认推荐）
3. 如果用户不满意，可重新展示所有 14 种风格列表
4. 确认后，记录选定的风格信息

### 4.2.5 应用风格到插图生成

在 Step 4.3 生成插图时：
1. 根据内容需要确定插图类型（comparison/ability/process）
2. 使用 Step 4.2 确认的风格色值
3. 组合生成最终插图

### 风格与插图类型组合示例

| 插图类型 | 风格主题 | 结果描述 |
|---------|---------|---------|
| comparison | 暗色科技风 | 深色霓虹对比图（左右两栏深色背景 + 霓虹边框） |
| ability | 孟菲斯波普风 | 孟菲斯风格能力卡片（几何装饰 + 鲜艳配色） |
| process | 商务深蓝风 | 商务风格流程图（深蓝主题 + 专业配色） |

---

### 内容要求

- 🎯 **思想有深度**：有洞察、有思考、有升华
- 🎯 **立意要高**：站在更高视角，挖掘本质
- 🎯 **观点要鲜明**：敢于表达立场，有独特见解

详见：[PROCESS_GUIDE.md#内容要求](specs/PROCESS_GUIDE.md#32-内容要求强制)

---

## 输出位置

**默认**：`./output/[文章slug]/`，在Agent的工作目录下，而不是skill的目录。

**可配置**：
- 调用参数：`--output ./my-output`
- 环境变量：`WECHAT_ARTICLE_OUTPUT_DIR`

---

## 详细规范

- **完整格式规范**：[FORMAT_SPEC.md](specs/FORMAT_SPEC.md)
- **完整流程指南**：[PROCESS_GUIDE.md](specs/PROCESS_GUIDE.md)
- **脚本工具说明**：[scripts/README.md](scripts/README.md)

---
> Source: [maple-yf/write-wechat-article](https://github.com/maple-yf/write-wechat-article) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
