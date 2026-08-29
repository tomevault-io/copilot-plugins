## deobf

> AI Agent 统一反混淆技能套件。一个调度 skill 加载 26 个专业反混淆/逆向子 skill，根据目标类型按需加载。

# AGENTS.md — deobf-all

AI Agent 统一反混淆技能套件。一个调度 skill 加载 26 个专业反混淆/逆向子 skill，根据目标类型按需加载。

## Project

- **类型**: Agent Skill 调度器（无编译产物，纯 Markdown/Shell）
- **语言**: Bash (install.sh), Batch (install.bat), Markdown (SKILL.md), MDX (docs/)
- **入口**: `deobf-all/SKILL.md` — 调度器本体，被 `npx skills` 或手动复制到 `~/.agents/skills/deobf-all/`
- **文档**: `docs/` — Mintlify 站点（docs.json + .mdx 页面），已推送至 https://github.com/zeij-drive/docs
- **许可证**: MIT

## Commands

```bash
# 一键全量安装（推荐）
curl -fsSL https://raw.githubusercontent.com/zeij-drive/deobf/master/install.sh | bash

# 安装（全局，macOS/Linux）
chmod +x install.sh && ./install.sh

# 安装（全局，Windows）
install.bat

# 安装到当前项目（不写全局目录）
./install.sh --local

# 预览安装（不实际执行）
./install.sh --dry-run

# Mintlify 本地预览（需 mintlify CLI）
cd docs && mintlify dev
```

无构建/测试命令 — 项目为纯配置/脚本类，不产出编译物。

## Architecture

```
deobf-all/              # 仓库根
├── deobf-all/SKILL.md  # 调度器 skill — 加载全部子 skill + 路由决策树
├── install.sh          # macOS/Linux 自动安装脚本
├── install.bat         # Windows 自动安装脚本
├── docs/               # Mintlify 文档站点
│   ├── docs.json       # 站点配置（导航、主题、锚点）
│   ├── *.mdx           # 页面（introduction, quickstart, skills/*, examples/* …）
│   └── logo/           # Logo SVG（待补充）
├── README_EN.md        # 项目说明（英文）
└── README.md           # 项目说明（中文）
```

**调度器技能层级**:
- **P0** (核心): `code-obfuscation-deobfuscation`, `ast-deobfuscation`
- **P1** (按需): `vm-and-bytecode-reverse`, `anti-debugging-techniques`, `symbolic-execution-tools`
- **P2** (辅助): `binary-protection-bypass`, `ctf-reverse`, `anti-reversing-techniques`, `deep-analysis`
- **P3** (扩展): `deobf-string`, `deobf-indirect`, `llvm-obfuscation`, `binary-lifting`, `deobfuscating-javascript-malware`, `deobfuscating-powershell-obfuscated-malware`, `binary-analysis-patterns`, `yara-rule-authoring`, `ghidra-headless`, `reverse-engineering-malware-with-ghidra`, `frida-17`, `radare2`

## Conventions

- 文档使用中文撰写（面向中文用户），代码/技术术语保留英文原文
- MDX 页面 frontmatter 必须包含 `title` 和 `description`
- 新增子 skill 需同步更新: `deobf-all/SKILL.md`（路由表）+ `install.sh`/`install.bat`（安装列表）+ `docs/docs.json`（导航）+ `docs/skills/`（参考页）
- 安装脚本使用 `npx skills add <repo> --skill <name> -g -y` 格式
- docs.json 中的 `YOUR_USERNAME` 和 `YOUR_INVITE` 是占位符，部署前须替换

## Notes

- 待补充: docs/logo/ 下的 SVG 文件
- 待补充: docs.json 中的 GitHub 用户名和 Discord 链接

---
> Source: [zeij-drive/deobf](https://github.com/zeij-drive/deobf) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-28 -->
