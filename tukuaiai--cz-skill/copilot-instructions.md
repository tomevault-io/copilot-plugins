## cz-skill

> │   └── demo-conversation-2026-04-08.md

# cz-skill

## 目录结构

```text
cz-skill/
├── AGENTS.md
├── LICENSE
├── README.md
├── SKILL.md
├── examples/
│   └── demo-conversation-2026-04-08.md
└── references/
    └── research/
        ├── 01-writings.md
        ├── 02-conversations.md
        ├── 03-expression-dna.md
        ├── 04-external-views.md
        ├── 05-decisions.md
        └── 06-timeline.md
```

## 文件职责

- `README.md`：GitHub 展示页，说明 skill 能力、效果、安装方式、来源与仓库结构。
- `LICENSE`：开源许可证，便于单独发布仓库。
- `SKILL.md`：最终可激活的 CZ 视角技能，负责角色规则、心智模型、决策启发式和表达 DNA。
- `examples/demo-conversation-2026-04-08.md`：实战对话示例，展示 skill 在多轮问题中的表现。
- `references/research/01-writings.md`：整理《币安人生》与原则附录中的系统性观点。
- `references/research/02-conversations.md`：整理回复、AMA、公开访谈式表达中的即兴思考方式。
- `references/research/03-expression-dna.md`：用推文数据统计表达风格、句式、词汇与互动偏好。
- `references/research/04-external-views.md`：记录外部人物、社区与争议视角，防止 skill 只剩自述。
- `references/research/05-decisions.md`：提炼重大决策、触发条件与可迁移的判断规则。
- `references/research/06-timeline.md`：维护时间线与最近动态，避免 skill 过时。

## 依赖边界

- `SKILL.md` 只消费 `references/research/` 的结论，不直接依赖外部原始文件。
- 外部原始资料主要来自《币安人生》、Hugging Face 数据集 `123olp/ersheng-cz-heyi-full` 与公开页面快照。
- 研究文档必须显式标注：哪些是 CZ 自述、哪些是公开行为、哪些是外部评价、哪些是推断。

## 设计约束

- 目标不是模仿口癖，而是提炼可运行的判断系统。
- 任何涉及法律、刑责、监管定性的内容都要保守表述，优先记录公开事实与边界。
- 对矛盾不做抹平处理，保留“重视合规”与“强调速度”之间的张力。

## 变更记录

- `2026-04-08`：基于《币安人生》、`ersheng-cz-heyi-full` 推文数据集与公开页面快照首次生成。
- `2026-04-08`：补齐 `README.md`、`LICENSE`、`examples/`，移除本地绝对路径，整理为可单独发布的仓库结构。

---
> Source: [tukuaiai/cz-skill](https://github.com/tukuaiai/cz-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-03 -->
