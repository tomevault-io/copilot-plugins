## awesome-claude-code

> > 本文件供 AI Agent（包括 Claude Code）和人类维护者参考。

# Awesome Claude Code — 仓库维护指南

> 本文件供 AI Agent（包括 Claude Code）和人类维护者参考。

---

## 1. 项目定位

本仓库是一份**精选的 Claude Code / AI Coding 生态资源清单**。

核心原则：

- **精选而非 exhaustive**：质量优先于数量
- **场景导向**：每个资源都要说明"什么时候用它"
- **点评而非罗列**：至少一句话说明价值
- **持续更新**：定期清理过时资源

---

## 2. 目录结构

```
awesome-claude-code/
├── README.md              # 首页
├── AGENTS.md              # 本文件
│
├── skills/                # Claude Code SKILL.md
├── mcp-servers/           # MCP servers
├── rules/                 # .cursorrules / CLAUDE.md 模板
├── cli-tools/             # CLI 工具
├── plugins/               # IDE 插件 / 编辑器扩展
├── prompts/               # Prompt 模板
├── workflows/             # AI Coding 工作流
├── eval/                  # 评测工具和方法
├── templates/             # 项目模板
└── references/            # 参考资料
```

---

## 3. 命名规范

- 目录：`kebab-case`
- Markdown 文件：`kebab-case`
- 资源条目：使用官方项目名

---

## 4. 内容格式

每个分类 README 中的条目建议格式：

```markdown
### [项目名称](链接)

**类型**: skill / mcp-server / cli / plugin / ...
**场景**: 一句话说明什么时候用它
**特点**: 1-3 个核心特点
**安装/使用**: `命令` 或 链接
```

示例：

```markdown
### [claude-code-eval-cli](https://github.com/vitoworleone/claude-code-eval-cli)

**类型**: eval framework  
**场景**: 想系统评估 Claude Code 在真实任务上的表现  
**特点**: 覆盖 Agent / Prompt / RAG 三层 eval，方法论优先  
**使用**: 查看 [README](https://github.com/vitoworleone/claude-code-eval-cli#readme)
```

---

## 5. 收录标准

### ✅ 收录

- 和 Claude Code 或 AI Coding 强相关
- 有实际使用场景
- 有公开文档或源码
- 社区有真实使用反馈

### ❌ 不收录

- 营销页面或空壳项目
- 停止维护超过一年
- 闭源且无文档
- 和 AI Coding 无关的通用工具

---

## 6. 更新流程

1. 新增资源时，先确认它符合收录标准
2. 放入正确分类
3. 按统一格式撰写条目
4. 在 README 的"精选推荐"中酌情更新

---

## 7. 版权与声明

- 本仓库只收录和点评第三方资源
- 引用资源时标注官方链接
- 不托管第三方源码

---

*最后更新: 2026-07-07*

---
> Source: [vitoworleone/awesome-claude-code](https://github.com/vitoworleone/awesome-claude-code) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-20 -->
