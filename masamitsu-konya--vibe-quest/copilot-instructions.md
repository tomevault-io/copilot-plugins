## vibe-quest

> - **`supabase db push`コマンドは絶対に実行しない**

# Vibe Quest プロジェクト固有の指示

## 重要な制約事項

### データベースマイグレーション
- **`supabase db push`コマンドは絶対に実行しない**
- マイグレーションファイルの作成はOK
- マイグレーションの適用は必ずユーザーが手動で行う
- 理由：マイグレーションファイルにエラーがあった場合、修正が困難になるため

### 開発ルール
1. マイグレーションファイルを作成したら、ユーザーに以下を伝える：
   - 作成したファイルのパス
   - 適用するコマンド: `supabase db push`
   - 変更内容の概要

2. データベース変更が必要な場合：
   - マイグレーションファイルを作成
   - ユーザーに確認を求める
   - ユーザーが手動で適用

## プロジェクト情報
- プロジェクト名: Vibe Quest
- Supabase Project Ref: ztaakebbntapjlqolkcx
- 主要技術: Flutter, Supabase, Riverpod

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/masamitsu-konya) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
