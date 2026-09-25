## light-skills

> 本仓库是公开科研全流程技能包，面向 Claude Code / Codex / OpenCode。请把 `skills/` 视为唯一 canonical 技能源；不要把维护者私人记忆、对话史、邮箱、未公开研究、db01-db09、MCP 配置或本地知识库内容写进仓库。

# Light-Skills 代理协作说明

本仓库是公开科研全流程技能包，面向 Claude Code / Codex / OpenCode。请把 `skills/` 视为唯一 canonical 技能源；不要把维护者私人记忆、对话史、邮箱、未公开研究、db01-db09、MCP 配置或本地知识库内容写进仓库。

## 客户端发现路径

三类客户端默认发现路径不同。项目级使用前先运行：

```powershell
$env:PYTHONUTF8="1"
python scripts\bootstrap_agent_skills.py --mode auto --force
```

该脚本把 `skills/` 安全映射到：

- `.agents/skills`：Codex / OpenCode open-standard 项目技能路径。
- `.claude/skills`：Claude Code 项目技能路径，也可被 OpenCode 读取。
- `.opencode/skills`：OpenCode 原生项目技能路径。

Windows 不能创建目录 symlink 时，脚本会回退为 copy，并写入 `.light_generated_link` 标记；修改 canonical `skills/` 后需重新运行 `--force` 刷新生成副本。脚本不会覆盖没有 Light 生成标记的用户目录。

## 开发纪律

- commit 只署作者本人；标题和正文写中文；不要添加 Co-Authored-By。
- 不要 push，除非用户明确要求。
- Python 命令前设置 `PYTHONUTF8=1`；临时产物放 `.upgrade/_e2e`。
- 用户面向脚本必须亲手跑 `--selftest` 到 exit 0，不能相信“已完成”报告。
- 动态事实、论文、DOI、star、链接、能力声明查不到就写 `unknown/待核查`。
- 论文数据图必须程序化生成；禁止 AI 生图冒充数据图。
- R/ggplot2 是增强路径，不是静默前提：缺 R 或缺包时，先给用户“继续 matplotlib 诚实降级 / 安装或配置 R / 提供 Rscript 路径”的选择；非交互自动流程默认降级并标 `degraded=True`，不得擅自安装。

## 快速自测

```powershell
$env:PYTHONUTF8="1"
python scripts\bootstrap_agent_skills.py --selftest
python scripts\bootstrap_agent_skills.py --check-only
python skills\light-figure\scripts\r_ggplot.py --selftest
```

---
> Source: [Light0305/Light-skills](https://github.com/Light0305/Light-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-24 -->
