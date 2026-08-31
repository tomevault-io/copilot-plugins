## old-agent-skills

> スキル名は原則として `service-target-action` の順で付けます。

# Agent Instructions

日本語で簡潔かつ丁寧に回答してください。

## Skill Naming Convention

スキル名は原則として `service-target-action` の順で付けます。

- `service`: `git` `github` `bun` `codex` `skill` のような対象領域
- `target`: `branch` `pr` `issue` `dependency` `skills` のような主対象
- `action`: `create` `review` `update` `description` のような操作内容

一覧を見た時に「どこで」「何に対して」「何をする」スキルかを判断できる名前にします。
README のグルーピングはスキル名の prefix ではなく、利用目的を優先して決めます。

## Skill Maintenance

- `SKILL.md` は原則として180行以内に収め、必須ルール・禁止事項・主要ワークフローは先頭150行以内に置く
- 長い例、詳細手順、CLI/APIサンプル、トラブルシュートは `references/` に用途別で分割する
- スキルの追加・変更時は、`SKILL.md`、`references/`、`scripts/`、プログラム内の import・外部コマンドを確認し、README の `External Dependencies` を同期する
- スキルの追加・削除時は、README 先頭の Shields.io スキル数バッジも同期する
- `.claude/skills/` 配下のリポジトリ保守専用スキルは品質検証対象に含めるが、README の `Available Skills` とスキル数バッジには含めない

ローカル検証:

```sh
bash .scripts/validate-skills.sh
```

この検証では、`SKILL.md` の行数、`references/` の参照切れ、README の `Available Skills` と実スキル一覧、スキル数バッジ、依存カラムの一致を確認します。

### スキル設計ルール

スキルのカテゴリ分類、命名規則、依存方向、責務境界、外部依存種別、構造規則、検証基準は [.rules/skill-rules.yaml](.rules/skill-rules.yaml) を正本とする。
全スキルのカテゴリ割り当てと依存関係は [.rules/skill-categories.yaml](.rules/skill-categories.yaml) に記録する。

#### カテゴリ一覧（依存方向: 上位→下位 のみ許可）

| カテゴリ | 概要 | 依存可能（他カテゴリ） |
|----------|------|----------|
| `orchestration` | 高次ワークフロー統括 | github, git, design, skill, tool, dependency |
| `github` | GitHubプラットフォーム操作 | git, design, tool |
| `git` | Gitローカル操作 | tool |
| `design` | 設計・計画支援 | —（leaf） |
| `skill` | スキル管理 | —（leaf） |
| `tool` | 独立ツール | —（leaf） |
| `dependency` | 依存パッケージ管理 | —（leaf） |

同一カテゴリ内のスキル間依存は §4.2 に従う（循環なし・汎用→特殊方向・正本への明記が条件）。

---
> Source: [u7chan/old-agent-skills](https://github.com/u7chan/old-agent-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-31 -->
