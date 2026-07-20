## blindspot-finder

> 本仓库有三份内容必须完全一致的 SKILL.md：

# blindspot-finder 项目说明

## 常驻纪律

### SKILL.md 三副本同步

本仓库有三份内容必须完全一致的 SKILL.md：

- `SKILL.md`（根目录，**以此为准**）
- `.claude/skills/blindspot-finder/SKILL.md`
- `.agents/skills/blindspot-finder/SKILL.md`

改动流程：**只改根目录那份**，再复制覆盖另外两份，然后跑校验：

```bash
diff <(cat SKILL.md) <(cat .claude/skills/blindspot-finder/SKILL.md)
diff <(cat SKILL.md) <(cat .agents/skills/blindspot-finder/SKILL.md)
```

两条命令都无输出（退出码 0）才算改完。后两份被 gitignore，`git diff`
不会暴露它们的不一致——漏同步不会报错，只会让实际加载的技能与仓库里
读到的版本悄悄分叉，所以校验这一步不能省。

---
> Source: [HeathTeng/blindspot-finder](https://github.com/HeathTeng/blindspot-finder) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-20 -->
