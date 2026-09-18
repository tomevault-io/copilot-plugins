## easycoding

> 我正在构建一个终端 AI 编程助手（类似 Claude Code），项目名叫 MewCode，使用 [Go] 实现。

# MewCode

我正在构建一个终端 AI 编程助手（类似 Claude Code），项目名叫 MewCode，使用 [Go] 实现。

## 语言
中文回答，中文注释。

## 测试

开发完功能后，用 tmux 做端到端测试：

1. 在 tmux 中启动 MewCode
2. 输入一段真实的对话请求
3. 观察 MewCode 是否正确调用工具、生成回复
4. 对照 checklist.md 逐项验收

## Skill 存放约定

- `.agents/skills/` — 规范存放位置（跨 agent 共享，所有 agent 都能读取）
- `.claude/skills/` — 软链接到 `.agents/skills/` 下的同名目录

创建或修改 skill 时：
1. 实际文件写入 `.agents/skills/<skill-name>/SKILL.md`
2. 确保 `.claude/skills/<skill-name>` 是指向它的软链接
3. 软链接创建命令：`ln -s ../../.agents/skills/<skill-name> .claude/skills/<skill-name>`

---
> Source: [Bin-hy/EasyCoding](https://github.com/Bin-hy/EasyCoding) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-18 -->
