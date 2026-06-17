## xiaolab-group-meeting-skill

> 生成 xiaolab 风格的组会 HTML deck，默认交付单个 HTML 文件。适用于用户要求“组会 PPT”“组会 deck”“xiaolab 风格汇报”“实验室组会 HTML 页面”时。


# Xiaolab Group-Meeting Deck

## 这个 Skill 做什么

生成一份可直接打开的 **单文件 HTML 组会 deck**。

默认产物是：
- 一个 `index.html`
- 如有配图，再配一个同级 `images/` 目录

不要把普通用户任务默认做成：
- `core/index.html`
- `components/index.html`
- 多份模板拆页

这些只属于仓库内部模板资源，不属于普通用户最终交付。

## 何时使用

当用户要：
- 做一份 xiaolab 风格组会汇报
- 做一份带 xiaolab / 湖南大学 / 超算中心品牌层的 HTML deck
- 把实验进展、论文阶段汇报、系统演示整理成可翻页网页 PPT

不用于：
- 泛化的商业 PPT
- 和 xiaolab 无关的展示页
- 模板仓库维护说明文档

## 默认工作流

### 1. 先对齐最少信息

如果用户没给完整大纲，先补齐这些信息：
- 受众是谁
- 汇报时长
- 有没有现成材料或截图
- 是否有必须出现的图表或数据

如果这些信息不影响开工，就做合理假设，不要为了小问题卡住产出。

### 2. 从单文件模板出发

把 `template/index.html` 复制到目标位置作为成品起点。

默认目标形态：
- `项目目录/ppt/index.html`
- `项目目录/ppt/images/...`

不要默认让用户去理解仓库里的模板组织。

### 3. 需要扩展布局时再参考组件页

`template/components/index.html` 只是布局参考页。

只有当默认模板不够用时，才从组件页借布局思路；最终仍然回写到用户的单个 `index.html` 里。

### 4. 保持可见页面干净

不要在成品页上留下：
- 模板说明文字
- 备注
- 占位提示语
- “核心页 / 参考页” 之类的元语言

### 5. 交付前做一轮快速质检

需要时读取：
- `references/component-patterns.md`
- `references/checklist.md`

至少确认：
- 最终交付仍是单个 HTML
- 品牌图标正常加载
- ESC 总览不泄漏备注
- 页面上没有占位符或教学文本

## 输出契约

默认输出契约只有一条：

**给使用者交付一个可直接打开的 HTML 页面。**

如果用户没有明确要求维护模板仓库，就不要把输出拆成多份模板文件。

---
> Source: [L1uYun/xiaolab-group-meeting-skill](https://github.com/L1uYun/xiaolab-group-meeting-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
