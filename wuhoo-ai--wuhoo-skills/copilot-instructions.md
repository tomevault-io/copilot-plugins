## wuhoo-skills

> Wuhoo 技能仓库。与 Agent 网关选型无关，可独立迭代。

# Skills Repository CLAUDE.md

Wuhoo 技能仓库。与 Agent 网关选型无关，可独立迭代。

仓库路径：`~/wuhoo-skills/`
通过软链接 `~/.openclaw/skills/` 加载。

## Skills 列表

### 企业级关键 Skills (wuhoo-*)
| Skill | 目录 | 用途 |
|-------|------|------|
| wuhoo-stock-deep-analysis | `wuhoo-stock-deep-analysis/` | Workflow B — 单股深度分析 |
| wuhoo-stock-trade | `wuhoo-stock-trade/` | Workflow C — 多市场交易执行 |
| wuhoo-trade-diagnose | `wuhoo-trade-diagnose/` | Workflow D — 持仓诊断与调仓 |
| wuhoo-news-rss | `wuhoo-news-rss/` | RSS 资讯引擎 |

### 交易相关 Skills
| Skill | 目录 | 用途 |
|-------|------|------|
| futu-api | `futu-api/` | 富途 OpenAPI 行情交易 (57 脚本) |
| install-futu-opend | `install-futu-opend/` | Futu OpenD 安装助手 |

### 选股 Skills
| Skill | 目录 | 用途 |
|-------|------|------|
| wuhoo-stock-pick | `wuhoo-stock-pick/` | A/HK/US 可配置因子选股 |

## Skill 开发规范

- 每个 skill 必须有 `SKILL.md` 文件
- SKILL.md 包含：用途描述、使用示例、依赖说明
- 运行时数据（data/, venv/）放入 .gitignore
- 修改 wuhoo-* skills 时需要特别谨慎，承担核心业务价值

## 软链接

```
~/.openclaw/skills/ → ~/wuhoo-skills/
```

openclaw.json 通过 `skills.load.extraDirs: ["~/.openclaw/skills"]` 加载。

---
> Source: [wuhoo-ai/wuhoo-skills](https://github.com/wuhoo-ai/wuhoo-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
