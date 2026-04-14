## thesis-agent

> 导师反馈处理工作流程


# Supervisor Feedback Workflow

## 工作模式

用户会提供：
1. **导师的修改意见**（中文或英文）
2. **对应章节的 LaTeX 源码**

我的任务：
1. 理解导师意见
2. 在 LaTeX 源码中进行针对性修改
3. 返回修改后的完整 LaTeX 代码

## 处理导师意见的流程

### Step 1: 分析导师意见
- 明确导师要求修改的具体内容
- 定位需要修改的段落或句子

### Step 2: 执行修改
- **内容修改**: 按导师要求调整论述、补充说明
- **语言修改**: 改进表达、修正语法
- **结构修改**: 调整段落顺序或逻辑

### Step 3: 输出结果
- 返回修改后的完整 LaTeX 代码
- 简要说明修改了什么

## ⚠️ 修改时的禁止事项

1. **不修改** `\cite{}` 等引用命令
2. **不修改** `\ref{}` 交叉引用
3. **不修改** figure/table/equation 环境
4. **不添加** 新的引用（除非导师明确要求）
5. **不删除** 现有引用

## 输出格式示例

```
### 导师意见 1: [意见内容]

**修改说明**: [简述如何修改]

**修改后代码**:
\`\`\`latex
[完整的修改后 LaTeX 代码]
\`\`\`
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/woaijiadanoo)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/woaijiadanoo)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
