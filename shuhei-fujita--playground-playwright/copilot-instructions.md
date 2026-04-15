## playground-playwright

> MECE Rules Index for Playwright / MECE ルール体系インデックス


# 📋 Playwright TypeScript ルール体系 / Playwright TypeScript Rules System

## ルールファイル構成（1次情報源） / Rule File Structure (Primary Source)

| ファイル / File                              | 内容 / Content                                                                                  | 適用範囲 / Scope                      | 更新頻度 / Update Frequency |
| -------------------------------------------- | ----------------------------------------------------------------------------------------------- | ------------------------------------- | --------------------------- |
| [📘 core.mdc](core.mdc)                       | プロジェクト基本情報・技術スタック / Project basics & tech stack                                | 全体 / All                            | 低 / Low                    |
| [🔒 security.mdc](security.mdc)               | 認証情報管理・セキュリティ基準 / Auth management & security standards                           | 全体 / All                            | 中 / Medium                 |
| [🎯 selectors.mdc](selectors.mdc)             | セレクター選択戦略・優先順位 / Selector strategy & priority                                     | テストコード / Test code              | 低 / Low                    |
| [🏗️ architecture.mdc](architecture.mdc)       | Page Object Model・構成規則 / Page Object Model & structure rules                               | 設計 / Design                         | 中 / Medium                 |
| [📊 quality.mdc](quality.mdc)                 | 品質基準・チェックリスト / Quality standards & checklist                                        | 全体 / All                            | 高 / High                   |
| [📁 test-outputs.mdc](test-outputs.mdc)       | テスト出力物管理・クリーンアップ / Test output management & cleanup                             | 運用 / Operations                     | 中 / Medium                 |
| [🧪 test-structure.mdc](test-structure.mdc)   | Given-When-Then・アサーション戦略 / GWT & assertion strategy                                    | テスト設計 / Test design              | 高 / High                   |
| [🎯 expect-strategy.mdc](expect-strategy.mdc) | expect配置戦略・Playwright公式準拠 / Expect placement strategy & Playwright compliant           | 実装 / Implementation                 | 中 / Medium                 |
| [📊 logging.mdc](logging.mdc)                 | ログシステムルール・セキュリティ・パフォーマンス / Logging system rules, security & performance | 実装・品質 / Implementation & Quality | 高 / High                   |
| [❓ project-faq.mdc](project-faq.mdc)         | プロジェクト固有Q&A・トラブルシューティング / Project-specific FAQ & troubleshooting            | 全体 / All                            | 高 / High                   |

## MECE分析結果 / MECE Analysis Results

### ✅ **Mutually Exclusive (相互排他性)** - 重複なし
- **core.mdc**: プロジェクト全体の基本情報・設定
- **security.mdc**: セキュリティ特化（認証情報・機密データ）
- **selectors.mdc**: セレクター戦略特化（要素選択方法）
- **architecture.mdc**: 設計パターン特化（Page Object・構造）
- **quality.mdc**: 品質基準特化（チェックリスト・メトリクス）
- **test-outputs.mdc**: 運用管理特化（出力物・クリーンアップ）
- **test-structure.mdc**: テスト設計特化（GWT・アサーション）
- **expect-strategy.mdc**: expect配置戦略特化（Playwright公式準拠）
- **logging.mdc**: ログシステム特化（ルール・セキュリティ・パフォーマンス）
- **project-faq.mdc**: Q&A特化（トラブルシューティング・ベストプラクティス）

### ✅ **Collectively Exhaustive (網羅性)** - 漏れなし
1. **📋 プロジェクト管理**: core.mdc
2. **🔒 セキュリティ**: security.mdc + logging.mdc（ログセキュリティ）  
3. **🎯 実装技術**: selectors.mdc + architecture.mdc + expect-strategy.mdc + logging.mdc（ログ統合）
4. **📊 品質保証**: quality.mdc + test-structure.mdc + logging.mdc（ログ品質）
5. **🛠️ 運用管理**: test-outputs.mdc
6. **❓ サポート**: project-faq.mdc

## ルール記述方針 / Rule Writing Policy

### 日本語・英語併記（必須） / Japanese-English Bilingual (Required)
- ✅ すべてのルールは日本語と英語の両方で記述 / All rules must be written in both Japanese and English
- ✅ 形式: `日本語 / English` / Format: `Japanese / English`
- ✅ LLMの理解精度向上のため / To improve LLM understanding accuracy
- ✅ 国際的な開発チームでの利用を考慮 / Consideration for international development teams

### 記述例 / Writing Example
```markdown
## セレクター戦略 / Selector Strategy
### 優先度（必須遵守） / Priority (Must Follow)
**理由 / Reason**: アクセシビリティ・セマンティクス重視 / Focus on accessibility and semantics
```

## ルール管理方針 / Rule Management Policy

### 1次情報源の原則 / Primary Source Principle
- ✅ `.cursor/rules/*.mdc` が**唯一の正式仕様** / is the **only official specification**
- ✅ 他のドキュメントは参照のみ（内容重複禁止） / Other docs are reference only (no content duplication)
- ✅ 矛盾時はルールファイル優先 / Rule files take priority in case of conflicts

### 更新プロセス / Update Process
1. ルールファイルを更新 / Update rule files
2. 関連コードに適用 / Apply to related code
3. README等の参照ドキュメントは変更不要 / No need to update reference docs like README

### 参照方法 / Reference Method
```markdown
詳細は [🎯 selectors.mdc](.cursor/rules/selectors.mdc) を参照
For details, refer to [🎯 selectors.mdc](.cursor/rules/selectors.mdc)
```

## 利用ガイド / Usage Guide

### 開発者向け / For Developers
- 新機能開発前: 関連ルールファイルを確認 / Before new feature development: Check related rule files
- コードレビュー時: ルール準拠をチェック / During code review: Check rule compliance
- 問題発生時: 該当ルールファイルを確認 / When issues occur: Check relevant rule files

### ルール更新者向け / For Rule Maintainers
- 関心事別にファイル分離 / Separate files by concern
- 重複内容の排除 / Eliminate duplicate content
- 参照しやすい構成の維持 / Maintain easily referenceable structure

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/shuhei-fujita) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
