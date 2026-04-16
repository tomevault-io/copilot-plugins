## seo-analytics-saas

> Windsurfルールファイルの作成と管理のためのガイドライン


<windsurf_rule_guideline>
# Windsurfルール作成・管理ガイドライン

## ルールファイルの基本構造

```markdown
# モデルディシジョンの場合
---
trigger: model_decision
description: "ルールの簡潔な説明（必須）"
---

# グロブの場合
---
trigger: glob
description: "ルールの簡潔な説明（必須）"
globs: "src/**/*.ts,lib/**/*.tsx"  # 適用対象のファイルパターン（必須）
---

# 常に適用する場合
---
trigger: always_on
description: "ルールの簡潔な説明（必須）"
---

# 手動適用の場合
---
trigger: manual
description: "ルールの簡潔な説明（必須）"
---
---

# ルールのタイトル

<rule_overview>
## 概要
ルールの目的と背景を説明します。
</rule_overview>

<rule_scope>
## 適用範囲
- このルールが適用される具体的なファイル種別や機能を記述
</rule_scope>

## XMLタグを使用したガイドライン例

<coding_guidelines>
- **命名規則**: キャメルケースを使用する
- **インデント**: スペース2つを使用する
- **コメント**: 複雑なロジックには必ずコメントを追加する
</coding_guidelines>

<performance_guidelines>
- 不要なレンダリングを避ける
- メモ化を適切に使用する
- 大きな配列の操作には最適化されたメソッドを使用する
</performance_guidelines>

<implementation_examples>
## 実装例
```typescript
// ❌ 違反例
function badExample() {
  // 非推奨の実装と理由
}

// ✅ 準拠例
function goodExample() {
  // 推奨される実装と利点
}
```
</implementation_examples>

<cautions>
## 注意事項
- 実装時の注意点や例外事項
</cautions>
```

<file_organization>
## ルールファイルの配置と命名規則

### 配置場所
すべてのルールファイルは以下の場所に配置します：
```
プロジェクトルート/
└── .windsurf/
    └── rules/
        └── your-rule-name.md
```

<naming_conventions>
### 命名規則
- **ケバブケース**（kebab-case）を使用
- 拡張子は `.md`
- 名前は目的を明確に表現する
  - 例: `use-typescript-strict.md`, `require-component-tests.md`
</naming_conventions>
</file_organization>

<trigger_modes>
## トリガーモードの詳細

| モード | 説明 | 使用ケース |
|------------|------|-----|
| `model_decision` | AIモデルが状況に応じて適用を判断 | コンテキスト依存のルールや複雑な判断が必要な場合 |
| `manual` | ユーザーが明示的に呼び出した場合のみ適用 | オプショナルなルールや特定の状況でのみ必要なルール |
| `always_on` | 常に自動的に適用される | プロジェクト全体に常に適用すべき基本ルール |
| `glob` | ファイルパターンに一致する場合に適用 | 特定のファイルタイプやディレクトリに限定したルール |
</trigger_modes>

<trigger_selection_guidelines>
### トリガーモードの選択ガイドライン

- **model_decision**: 複数の要因を考慮した高度な判断が必要な場合に使用します。例えば、コードの文脈やプロジェクトの状態に基づいて適用すべきか判断する場合。

- **manual**: ユーザーが明示的に呼び出す必要があるルールに使用します。例えば、パフォーマンス最適化や特定のコーディングスタイルの適用など。

- **always_on**: プロジェクト全体に常に適用すべき基本的なルールに使用します。例えば、コーディング規約やベストプラクティスなど。

- **glob**: 特定のファイルパターンに一致する場合に適用するルールに使用します。例えば、特定のフレームワークやライブラリに関連するファイルにのみ適用するルール。
</trigger_selection_guidelines>

<metadata_fields>
## メタデータフィールドの詳細

| フィールド | 説明 | 必要なトリガーモード | 例 |
|------------|------|-----|-----|
| `trigger` | ルールの適用タイミング（必須） | 全て | `"model_decision"`, `"manual"`, `"always_on"`, `"glob"` |
| `description` | ルールの簡潔な説明（必須） | 全て | "TypeScriptの厳格モードを使用する" |
| `globs` | 適用対象のファイルパターン（カンマ区切りの文字列） | glob | `"src/**/*.ts,lib/**/*.tsx"` |

<important_note>
**注意**: `alwaysApply`は使用しません。代わりに`trigger: always_on`を使用します。
</important_note>
</metadata_fields>

## ルール作成のベストプラクティス

<best_practices>
- **明確な目的**: 各ルールは単一の明確な目的を持つべき
- **具体的な例**: 良い例と悪い例を必ず含める
- **適用範囲の最小化**: 必要なファイルのみを対象とする
- **ドキュメント化**: 理由と背景を明確に説明する
- **一貫性**: 他のルールと矛盾しないようにする
- **globsの正しい形式**: 複数のファイルパターンは配列ではなく、単一の文字列にカンマ区切りでまとめる
- **XMLタグの活用**: 関連するルールをXMLタグでグループ化する
- **簡潔さ**: ルールは簡潔かつ具体的に記述する（6000文字以内）
</best_practices>

<globs_format>
## globsフィールドの正しい書き方

```markdown
# 間違った書き方（配列形式）
globs:
- "src/**/*.ts"
- "lib/**/*.tsx"

# 正しい書き方（カンマ区切りの単一文字列）
globs: "src/**/*.ts,lib/**/*.tsx"
```

<important_note>
**重要**: Windsurfではglobsフィールドは必ずカンマ区切りの単一文字列として指定する必要があります。配列形式では機能しません。
</important_note>
</globs_format>

<application_priority>
## ルール適用の優先順位

<priority_order>
- `trigger: always_on` のグローバルルール
- `trigger: glob` で特定のファイルタイプに対するルール
- `trigger: model_decision` のコンテキスト依存ルール
- `trigger: manual` の手動適用ルール
</priority_order>
</application_priority>

<character_limits>
## 文字数制限

<limitations>
- 各ルールファイルは6000文字以内に収める
- グローバルルールとワークスペースルールの合計は12,000文字以内
- 制限を超える場合、グローバルルールが優先され、超過分は切り捨てられる
</limitations>
</character_limits>

</windsurf_rule_guideline>

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/inabaphoto) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
