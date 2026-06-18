## daisan-kazoku-prompt-improver

> このリポジトリには2つの主要な機能があります：

# このリポジトリでの開発ルール

## プロジェクト構成

このリポジトリには2つの主要な機能があります：

1. **データラベリングWebアプリ** - gedokun投稿にラベルを付けるスワイプUI
2. **プロンプト評価システム** - AIモデレーションの精度を測定・改善

詳細は [README.md](README.md) を参照してください。

## ドキュメント管理ルール

### 作業メモ・進捗管理

実装タスクの進捗メモや設計メモは `docs/` 配下に管理してください：

- **作業メモ**: `docs/notes/` に追加
  - 例: `docs/notes/feature-xxx.md`, `docs/notes/bug-fix-yyy.md`
  - 一時的なメモや作業記録はここに

- **UI開発の進捗**: `docs/ui/` に追加
  - 例: `docs/ui/progress.md`, `docs/ui/design-decisions.md`

- **データセット**: `docs/dataset/` に追加
  - Firebase exportやその他のデータファイル

### プロンプト評価システムのドキュメント

評価システムに関するドキュメントは専用のディレクトリで管理：

- **実行ガイド**: `docs/evaluation/README.md`
  - 評価の実行方法、分析フロー、改善パターンなど
  - **新しい分析手法を発見したらここに追記してください**

- **評価結果の履歴**: `docs/evaluation/RESULTS.md`
  - 各バージョン（v1, v2, v3, v4...）の評価結果
  - 改善効果の分析、問題点の考察
  - **評価を実行したら必ずここに結果を追記してください**
  - **新しいプロンプトバージョンを作成したら改善内容を記録してください**

- **評価結果ファイル**: `docs/evaluation/results/`
  - `summary_*.json`, `errors_*.csv`, `evaluation_*.json`
  - 自動生成されるファイル（手動編集不要）

- **評価用データ**: `scripts/evaluation/data/`
  - `gedokun_posts_with_judgment.csv` - 人間が判定したデータ

### ドキュメント作成時の注意点

- 一時的なメモは `docs/notes/` に
- 正式なドキュメントは適切なディレクトリに（`docs/ui/`, `docs/evaluation/` など）
- プロンプト評価の分析結果は必ず `docs/evaluation/RESULTS.md` に記録
- 分析方法の改善やTipsを発見したら `docs/evaluation/README.md` に追記

## 型チェック

コードを変更したら必ず型チェックを実行してください：

```bash
npx tsc --noEmit
```

エラーがある場合は修正してから作業を完了としてください。

## コーディングルール

### 基本方針

- コードの変更は、既存のスタイル（TypeScript + React + Tailwind）に合わせて最小限に行ってください
- Supabase など外部サービスとの連携が必要な場合でも、まずはダミーデータやモックで UI を確認できるようにしてから、本番の連携を実装してください
- 新機能は段階的に実装し、各ステップで動作確認を行ってください

### ファイル命名規則

- コンポーネント: PascalCase（例: `SwipeCard.tsx`）
- ユーティリティ: camelCase（例: `formatDate.ts`）
- 型定義: PascalCase（例: `types.ts` 内の `PostData`）

## プロンプト評価の作業フロー

新しいプロンプトバージョンを作成・評価する際の標準フロー：

1. **エラー分析**: `docs/evaluation/results/errors_vN_*.csv` を分析
2. **プロンプト改善**: `scripts/evaluation/prompts.ts` に新バージョンを追加
3. **評価実行**:
   ```bash
   # 初回評価（素早く確認）
   pnpm tsx scripts/evaluation/run.ts --version vN+1

   # 詳細評価（ランダムモード推奨）
   pnpm tsx scripts/evaluation/run.ts --version vN+1 --random
   ```
   - **重要**: ランダムモードを使用して偏りを防ぐ
   - 複数回評価して平均的な精度を確認することを推奨
4. **結果記録**: `docs/evaluation/RESULTS.md` に結果を追記
   - 指標（Precision, Recall, F1 Score）
   - 使用したデータ選択モード（デフォルト/ランダム/範囲指定）
   - 改善効果
   - 残っている問題点
   - 改善内容の詳細
5. **分析方法の更新**: 新しい知見があれば `docs/evaluation/README.md` に追記

詳細な分析方法とデータ選択モードの使い方は [docs/evaluation/README.md](docs/evaluation/README.md) を参照してください。

## 変更履歴管理

- 新しいルールや運用方法を追加したい場合は、この `AGENTS.md` を更新してください
- 重要な変更は `docs/MIGRATION.md` や関連ドキュメントにも記録してください
- プロジェクトの歴史を追える形でドキュメントを維持してください

---
> Source: [daisan-kazoku/daisan-kazoku-prompt-improver](https://github.com/daisan-kazoku/daisan-kazoku-prompt-improver) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
