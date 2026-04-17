## designdocs-sample-fukazawasoftwaredevelopment-com

> このプロジェクトでユーザーストーリーを作成する際の標準ルールです。

# ユーザーストーリー作成ルール

このプロジェクトでユーザーストーリーを作成する際の標準ルールです。

## 基本ルール

### 1. ファイル保存場所
- ユーザーストーリーファイルは [docs/design/source/user_story/](mdc:docs/design/source/user_story) ディレクトリに保存
- ファイル名は内容を表す分かりやすい名前（例：`shopping_cart.rst`, `user_login.rst`）

### 2. テンプレートの使用
- [docs/design/source/user_story/template.rst](mdc:docs/design/source/user_story/template.rst) をコピーして使用
- テンプレートの構造を維持し、全セクションを埋める
- **Agile形式**を採用：「As a / I want / So that」+ 「Given-When-Then」

### 3. インデックスファイルへの追加
- [docs/design/source/user_story/index.rst](mdc:docs/design/source/user_story/index.rst) の `.. toctree::` セクションにリンクを追加
- ファイル名は拡張子（`.rst`）を除いて記載
- リンクの順序は機能の重要度や作成順を考慮
- `template.rst` は最後に配置

### 4. 用語・設計の統一
- 新しい用語は [docs/design/source/domain_model.rst](mdc:docs/design/source/domain_model.rst) に定義を追加
- 専門用語、システム固有の概念、業務用語は必ず定義
- ドメインモデルの用語と一貫性を保つ

## Agile形式のユーザーストーリー

### 基本構造
```
As a [アクター]
I want [具体的な機能・操作]
So that [価値・目的]
```

### 受け入れ条件（Given-When-Thenシナリオ）
```
シナリオ1: [成功パターン]
Given [前提条件]
When [アクション]
Then [期待される結果]

シナリオ2: [異常パターン1]
Given [前提条件]
When [アクション]
Then [期待される結果]

シナリオ3: [異常パターン2]
Given [前提条件]
When [アクション]
Then [期待される結果]
```

## テンプレート構造

必須セクション：

1. **タイトル**
   - issue番号と要求内容を明記
   - 簡潔で分かりやすい名前

2. **概要**
   - GitHubのissueへの参照
   - 背景と目的の説明

3. **ユーザーストーリー**
   - Agile形式（As a/I want/So that）のリスト
   - アクター、機能、価値を明確に

4. **受け入れ条件**
   - 3つのGiven-When-Thenシナリオ
   - 成功パターンと異常パターン

5. **ビジネス価値**
   - 機能実現による価値と影響
   - 定量的・定性的な効果

6. **制約・前提条件**
   - 技術的制約
   - 業務制約
   - 前提条件

## Git運用

### コミットタイミング
- 新規ユーザーストーリー作成時
- 既存ストーリーの重要な更新時
- インデックスファイル更新時

### コミットメッセージ形式
```
feat(user-story): [機能名] - [概要]
```

例：
- `feat(user-story): add shopping cart story`
- `feat(user-story): update login acceptance criteria`

## 品質チェックリスト

### 作成時チェック
- [ ] GitHubのissueと内容が一致している
- [ ] Agile形式で記述されている
- [ ] 受け入れ条件が具体的で測定可能
- [ ] ドメインモデルの用語と一貫性がある
- [ ] 3つのシナリオが記述されている
- [ ] インデックスにファイルリンクが追加されている

### レビューポイント
- [ ] ユーザーストーリーが価値ベースで記述されている
- [ ] 受け入れ条件が実装・テスト可能
- [ ] 制約・前提条件が明確
- [ ] 他の設計文書との整合性
- [ ] ビジネス価値が明確に示されている

## インデックスファイル運用

### 基本構成
```rst
ユーザーストーリー 一覧
============================================

.. toctree::
   :maxdepth: 1

   [ユーザーストーリーファイル名]
   template
```

### 追加手順
1. 新ファイルを作成
2. `toctree` に追加（拡張子なし）
3. 重要度順に並べ替え
4. `template` は最後に維持

## 関連ルール

- [ドメインモデル作成ルール](mdc:.cursor/rules/domain_model.mdc)
- [ユースケース作成ルール](mdc:.cursor/rules/usecase.mdc)
- [プロジェクト設計ワークフロー](mdc:.cursor/rules/workflow.mdc)
- [画面遷移図作成ルール](mdc:.cursor/rules/screen.mdc)
- [データベース設計ルール](mdc:.cursor/rules/database.mdc)
- [DDD設計ルール](mdc:.cursor/rules/ddd.mdc)
- [バッチ処理設計ルール](mdc:.cursor/rules/batch.mdc)
- [メール定義書作成ルール](mdc:.cursor/rules/mail.mdc)
- [Git運用ルール](mdc:.cursor/rules/git.mdc)
- [Mermaid記法ルール](mdc:.cursor/rules/mermaid.mdc)

## 設計原則

### 1. ユーザー中心設計
- ユーザーの視点で価値を記述
- 具体的な操作と期待結果を明確に
- 実際のユースケースに基づく

### 2. 測定可能な受け入れ条件
- 具体的な条件と期待結果
- テスト可能な形式
- エッジケースの考慮

### 3. トレーサビリティ
- GitHubのissueとの紐付け
- 関連する設計文書への参照
- 変更履歴の管理

### 4. 一貫性の維持
- ドメインモデルとの用語統一
- テンプレート構造の遵守
- 命名規則の統一

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/MasaruFukazawa) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
