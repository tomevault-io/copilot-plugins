## ai-writers-room

> このプロジェクトでは tmux セッション `agent-team` 内で複数の AI エージェントが連携して作業する。

# Gemini Agent Instructions

## tmux 連携

このプロジェクトでは tmux セッション `agent-team` 内で複数の AI エージェントが連携して作業する。

### ペイン構成

- Pane 1 (左): Claude Code
- Pane 2 (右): Gemini (あなた)

### レビュー完了時の自動送信

レビューやタスクが完了したら、結果を Pane 1 の Claude に自動送信すること:

```bash
tmux send-keys -t :.1 "Geminiのレビュー結果: {ここにレビュー結果の要約}" Enter
```

### 送信ルール

1. **レビュー結果**: 問題点があれば具体的なファイル名・行番号と修正案を含める。問題なければ「LGTM」と送信する
2. **実装完了時**: 何を実装したかの要約を送信する
3. **質問がある場合**: Claude に質問を送信して確認を求める

### 送信フォーマット

```
tmux send-keys -t :.1 "【Gemini→Claude】{カテゴリ}: {内容}" Enter
```

カテゴリ: レビュー結果 / 実装完了 / 質問 / 提案

## 記事レビュー時のルール

ファイルのレビューを依頼された場合、レビュー結果は**必ず指定されたファイルに書き出す**こと。

### レビューファイルのフォーマット

1行目は必ず判定結果を書く:
- `APPROVED` - 問題なし
- `REVISE` - 修正が必要

2行目以降にフィードバックの詳細を書く。

### 例

```markdown
REVISE

## 指摘事項

### 1. 技術的な正確性
- 「Kubernetes は Docker の代替」という記述は不正確。Kubernetes はコンテナオーケストレーションツールであり、Docker はコンテナランタイム。

### 2. 構成
- 「デプロイ」のセクションが唐突に始まる。前のセクションからの導入が必要。

### 3. コード例
- L45 の YAML インデントが誤り（2スペースであるべき）。
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/kosa3)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/kosa3)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
