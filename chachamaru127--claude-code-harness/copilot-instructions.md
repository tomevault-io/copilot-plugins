## skill-subagent-usage

> 通常タスクで該当 Skill/Subagent を積極的に探して使う


# Skill と Subagent を積極活用する

## 使い分け

- **Skill**: 専門知識が必要なタスク → 作業開始前に `SKILL.md` を読み、手順/制約をそのまま適用する。宣言だけで終わらせない。
- **Subagent**: 独立コンテキストが有効なタスク（リファクタリング・レビュー・広範囲探索など）、または並列実行したい場合に委任する。

※ 両者は併用可。Skill の知識を Subagent に渡して実行することもある。
※ 小さいタスクで該当する Skill/Subagent がない場合は、通常フローで進める。

## 利用時は明示

使用する Skill/Subagent と理由は1行で明記してから進める。

---
> Source: [Chachamaru127/claude-code-harness](https://github.com/Chachamaru127/claude-code-harness) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-20 -->
