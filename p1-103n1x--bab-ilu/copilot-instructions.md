## bab-ilu

> > This file is read by Claude Code and other AI agents at session start.

# Bab-ilu — Agent Project Guide (v2.1)

> This file is read by Claude Code and other AI agents at session start.
> It tells agents how the project is organized and what conventions are non-negotiable.
> Authority source: `PRD-v2.1-zh.md` (especially §0.5 theoretical foundation, locked).

---

## 0. 理论基础（硬约束 · 每次会话前置词）

> 本节同步自 `PRD-v2.1-zh.md §0.5.2 + §0.5.3`。PRD 为权威源；若此处与 PRD 漂移，以 PRD 为准。修订本节需要重新对齐控制论与晚期维特根斯坦的理论基础。

### 核心价值命题（写死）

> **Bab-ilu 不是一个"知道规则"的系统，是一个"从实践中长出规则"的系统。**

- **控制论**（Wiener / Ashby / Beer）给"**如何从实践中长**"的工程实现——闭环、自适应、变异度匹配、晚绑定。
- **晚期维特根斯坦**（《哲学研究》——语言游戏 / 家族相似性 / 生活形式 / 规则跟随悖论）给"**为什么必须从实践中长**"的哲学依据。

**这两者不是并列的两个参考，是同一世界观的工程侧和哲学侧。** 所有与此核心命题冲突的设计决策都被拒绝，不论它在局部是否"更方便"。原则之间若产生冲突，回溯此命题决裁。

### 五条可操作原则

| # | 原则 | 具体含义 |
|---|---|---|
| 1 | **晚绑定**（late binding） | 任何能延迟的全局决策都延迟；优先收集 variety，再归纳 |
| 2 | **反本质** | Lens schema 松散，允许每个 lens 有独特字段；不强制共享结构（家族相似性） |
| 3 | **闭环优先** | 工程上先保证真实闭环（accept/reject 持久化 / 自适应阈值 / 反向提示词 / 跨会话学习），再考虑功能扩展 |
| 4 | **Variety 匹配**（Ashby） | Lens 必须能随用户 variety 成长。LENS EVOLUTION 是 Ashby 定律的工程实现 |
| 5 | **实践优于规则** | 不做"什么是好 lens"的教条；展示美学作为范例 + 邀请社区贡献，标准在实践中涌现 |

### 对 Karpathy 原教旨的必守点

这两处是 Karpathy 原文明示、但 Bab-ilu v2.1 尚未落地的核心机制。任何未来 PR 触碰 `/ask` 或 `/lint` 时，必须朝这两个方向推进而非偏离：

- **`/ask` 回存**——*"good answers can be filed back into the wiki as new pages... your explorations compound in the knowledge base just like ingested sources do."* 每次 `/ask` 应询问归档为 `wiki/syntheses/<slug>.md`。
- **`/lint` 主动建议**——*"the LLM is good at suggesting new questions to investigate and new sources to look for."* `/lint` 不只做局部合规检查，还要给出矛盾检测、过时检测、素材请求。

### 会话启动顺序（加入此节后更新）

每次会话启动时，agent 必须按以下顺序加载上下文：

1. **本节 §0（理论基础）** —— 绝不跳过。所有后续决策都须回溯这五条原则。
2. `PRD-v2.1-zh.md §0.5` —— 本节的权威来源与扩展说明。
3. `schema.md` —— 数据契约。
4. 下文项目特定约定（`## What Bab-ilu Is` 及之后）。

---

## What Bab-ilu Is

Bab-ilu is **a self-growing LLM-Wiki with pre-installed discipline-specific lenses**. It extends Karpathy's LLM-Wiki with two structural additions: a **pre-installed lens slot** (currently occupied by the aesthetic-warburg lens as the stress-test reference implementation) and a **user-space evolving lens layer** (Zettelkasten fallback + LENS EVOLUTION that grows action-lenses from user behavior). See `PRD-v2.1-zh.md §0.5` for the theoretical foundation locked as hard constraint.

Users drop source material into `raw/`; agents build `wiki/` (prose + wikilinks) and `.agent/graph/` (ontology + inference memory). Material type routes to the matching lens (visual-art → aesthetic; other → Zettelkasten + evolution observation). `/gap` surfaces emergent structures per active lens (Pathosformel / Patterns / concept-clusters / …).

Full product spec: `PRD-v2.1-zh.md` (Chinese, authoritative) / `PRD.md` (English, to be produced pre-open-source).
Schema contract: `schema.md`.
Algorithm spec: `.agent/spec/gap-algorithm.md`.

**Note**: Hard rules below were written in v2.0 when Bab-ilu was aesthetic-only. Some are **universal** (three-layer architecture, atomic writes, dual anchor principle) and apply to all lenses. Others (Panofsky three-layer contract, Pathosformel prohibitions) are **aesthetic-lens-specific** and only bind when the active lens is `aesthetic-warburg`. A full v2.1 rewrite of this file is deferred; for now, apply aesthetic-specific rules only under the aesthetic lens.

## Three-Layer Architecture (non-negotiable)

| Layer | Directory | Owner | Mutability |
|-------|-----------|-------|------------|
| Raw sources | `raw/` | User | Immutable — never modify |
| Human knowledge | `wiki/` | LLM writes; user curates | Read in Obsidian |
| Agent memory | `.agent/` | LLM owns | Hidden from Obsidian |

## Vault Language

Set at `/genesis` in `vault_language:` field below. All user-facing output uses this language.

```yaml
vault_language: zh     # this is the SOURCE project; individual user vaults override
```

Generated prompts for AI models remain English regardless.

## Hard Rules

1. **Never modify `raw/`.** If something seems wrong, surface to user.
2. **Read `schema.md` before writing in `wiki/`.**
3. **Atomic writes only.** Never partial.
4. **Wikilinks in prose; not in frontmatter.** Frontmatter minimal (≤3 fields typical).
5. **Section markers:** Auto-generated regions use `<!-- llm:section-start <name> -->` / `<!-- llm:section-end <name> -->`. User edits outside markers preserved on regenerate.
6. **Dual anchor:** AAT/Iconclass/ULAN in both prose footnote AND `.agent/graph/anchors.md`. `/lint` checks consistency.
7. **Panofsky three-layer contract:** `/taste` and `/prompt` output visibly separates pre-iconographic / iconographic / iconological.

## Commands (8 total)

| Command | Purpose |
|---------|---------|
| `/genesis [--seeded]` | Initialize vault; 4 interactive questions; optional seed-kit import |
| `/ingest <path\|query>` | Ingest raw material → create work/motif pages + update ontology |
| `/taste <image\|file>` | Single-input analysis; propose motifs (no commit without user approval) |
| `/gap [--tier=1\|2\|all]` | Run gap analysis → generate questions + todos |
| `/ask <question>` | Query vault; synthesize answer with citations |
| `/prompt <target>` | Generate AI-model prompts (optional downstream) |
| `/distill <source>` | Cross-language cognitive reconstruction of foreign source |
| `/lint [--rebuild-graph]` | Health check; verify wiki/ontology consistency |

Legacy v1.4 academic skills (18) moved to `.agent/skills/legacy-academic/` — opt-in, not on the main path.

## Prohibited

- Reintroducing v1.4 fields (`panofsky_layer`, museum-specific ID fields as frontmatter, `warburg_panel:` field)
- Manually creating Pathosformel or Topos pages (they emerge from `/gap`)
- Clustering aggregate nodes (pathosformel, topos) in subsequent `/gap` runs
- Translating foreign sources instead of cognitive reconstruction
- Writing to `.agent/graph/*` as anything other than append-only (retractions are additional lines)

## When Things Go Wrong

- Broken frontmatter → fix forward, surface to user
- Missing AAT resolution → record `null` + candidates in `.agent/state/vocabulary_resolution.md`; soft-prompt after 14-day grace
- User edits inside `<!-- llm:section-start -->` markers → respect intent; `/lint` flags for reconciliation
- Partial `/ingest` failure → resumable via `.agent/state/ingest-progress.md`

## Session Start Checklist

Before any tool call:
1. Read `schema.md` (≤200 lines, takes a few seconds)
2. Glance at `vault_language:` above
3. Note which command the user's request maps to
4. Check `.agent/state/` for any pending todos or uncertainty flags

---
> Source: [P1-103n1x/bab-ilu](https://github.com/P1-103n1x/bab-ilu) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-23 -->
