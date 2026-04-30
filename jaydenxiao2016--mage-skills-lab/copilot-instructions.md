## mage-skills-lab

> 本仓库是 **mage-* Cursor Skills** 聚合：每个技能在 `mage-skills/<名称>/`，入口文档为 **`SKILL.md`**。

# Agent 说明（Cursor / AI）

本仓库是 **mage-* Cursor Skills** 聚合：每个技能在 `mage-skills/<名称>/`，入口文档为 **`SKILL.md`**。

## 环境变量（必读）

- **规则全文**：`.mage-skills/README.md`（合并顺序、单 skill / 整仓两种用法、`MAGE_SKILLS_ROOT`）。
- **实现**：`.mage-skills/load-env.cjs`（脚本经 `load-env-bootstrap.cjs` 或各 skill 的封装调用）。

**给人配置时可以二选一：**

1. **整仓**：仓库根 `.mage-skills/.env`（模板 `.mage-skills/.env.example`）。  
2. **只装一个 skill**：在该技能根目录（与 `SKILL.md` 同级）复制 **`.env.example` → `.env`** 填写即可。

另可选 **`~/.mage-skills/.env`** 作本机共用。`.env` 勿提交 Git。

## 执行脚本前

在技能目录或仓库子目录下运行一般即可；若工作目录不在仓库内，可设 **`MAGE_SKILLS_ROOT`** 指向仓库根。

可选自检：`node .mage-skills/check-env.mjs`（在完整仓库内）。

---
> Source: [jaydenxiao2016/mage-skills-lab](https://github.com/jaydenxiao2016/mage-skills-lab) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-25 -->
