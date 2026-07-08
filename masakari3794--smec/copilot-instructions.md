## smec

> This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.

# AGENTS.md

This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.

## 概要

中小企業診断士 第一次試験の学習ノートリポジトリ。科目ごとに Markdown ファイルでまとめている。

## リポジトリ構造

```
第一次試験/
  企業経営理論/   # 戦略論・組織論・マーケティング等のノート
  財務・会計/     # 会計・財務・税務等のノート
skills/
  exam-question-drill/  # 演習問題生成スキル
    SKILL.md            # スキルの動作仕様
    references/
      past-exam-style.md  # 問題設計パターン集
```

## ノートの形式規約

- 各ファイルは `## 見出し` → `### 小見出し` → `*` 箇条書きの階層で記述
- ファイル名は英語ケバブケース（例: `5フォース分析.md`, `brand-management.md`）
- 内容はすべて日本語

## スキル

`/exam-question-drill` — 指定した科目・ファイルから本番レベルの演習問題を1問ずつ出題する。科目未指定時は確認を1回行う。

## PR ワークフロー

- PR は必ず upstream（`masakari3794/SMEC`）の `main` ブランチへ向けて作成する。
- 作業ブランチは作成せず、常に `main` ブランチ上で直接編集・コミットを行い、そのまま PR を作成する。

---
> Source: [masakari3794/SMEC](https://github.com/masakari3794/SMEC) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
