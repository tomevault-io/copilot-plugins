## zhiforge

> 将知识库炼化为知乎回答的 Claude Code Skill。

# 知炼 (ZhiForge)

将知识库炼化为知乎回答的 Claude Code Skill。

## 配置文件

**重要：** 使用前请先配置 `.claude/settings.json`：

```json
{
  "knowledge_base": "/path/to/markdown/files", // 知识库目录（存放 .md 文件）
  "blog_url": "https://your-blog.github.io",   // 可选，用于文章结尾引导
  "author": { ... },                            // 作者信息
  "expertise": [ ... ],                         // 专业领域
  "high_match_topics": [ ... ]                  // 高匹配度话题
}
```

## 当前配置

配置详情见 `.claude/settings.json`

## 博客文章索引（用于匹配知乎回答）

### AI / vibe-coding 系列
| 文件名 | 主题 | 适合回答的问题类型 |
|--------|------|-------------------|
| `2025-09-27-orion-vibe-coding-detailed.md` | AI 辅助完成的开源工程范本 | AI 编程实践、工程化 |
| `2026-01-27-git.md` | Git 生存技能（夯） | Git 使用、版本控制 |
| `2026-01-28-抄.md` | 抄出你的初始系统 | AI 编程入门、Docker |
| `2026-01-30-学.md` | 学习技术栈组合 | 技术选型、学习路径 |
| `2026-01-27-软件正在变成对话的副产品.md` | AI 对软件开发的影响 | AI 趋势、行业变化 |
| `2026-01-28-敬畏系统.md` | 系统思维 | 架构、工程思维 |
| `2026-01-25-半天上线gentskill.work...md` | AI 快速开发实战 | AI 编程效率、实战案例 |
| `2025-11-13-写在ai-shifu的v1.0.3.md` | AI-Shifu 项目 | 开源项目、AI 教育 |
| `2025-10-20-AI在变坏.md` | AI 发展思考 | AI 伦理、趋势 |
| `2026-02-27-my-lobster-came-alive.md` | OpenClaw 本地 AI Agent 实战 | AI Agent、本地部署、OpenClaw |

### 技术深度系列
| 文件名 | 主题 |
|--------|------|
| `2022-10-*-[Java性能优化实践]*.md` | JVM 性能优化（2-12章） |
| `2019-*-*.md` | OpenResty / Lua 开发 |

## 使用说明

1. **查看邀请** - `/check-zhihu`
2. **草拟回答** - `/draft-answer`
3. **发布文章** - `/publish-zhihu`
4. **全自动流程** - `/auto-zhihu`

## 依赖

**OpenClaw 模式（推荐）：**
- OpenClaw browser 工具（内置，无需额外安装）
- 知乎账号已登录（browser profile "openclaw"）

**Claude Code 模式：**
- Chrome MCP Server（用于操作知乎页面）
- 知乎账号已登录（在 Chrome 中）

---
> Source: [yfge/zhiforge](https://github.com/yfge/zhiforge) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-03 -->
