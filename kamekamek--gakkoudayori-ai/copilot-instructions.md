## gakkoudayori-ai

> このプロジェクトでは一貫したドキュメント管理を行うため、以下のルールに従ってください。

# ドキュメント管理ルール

このプロジェクトでは一貫したドキュメント管理を行うため、以下のルールに従ってください。

## 命名規則

### ファイル命名フォーマット
```
{ナンバー}_{カテゴリ}_{タイトル}.md
```

- **ナンバー**: 2桁数字（01, 02, 03...）
- **カテゴリ**: 大文字英語（REQUIREMENT, DESIGN, SPEC等）
- **タイトル**: アンダースコア区切り英語

### カテゴリとナンバリング体系

| 範囲 | カテゴリ | 説明 | 例 |
|-----|----------|------|-----|
| 01-09 | REQUIREMENT | 要件定義 | [01_REQUIREMENT_overview.md](mdc:docs/01_REQUIREMENT_overview.md) |
| 10-19 | DESIGN | 設計書 | [10_DESIGN_color_palettes.md](mdc:docs/10_DESIGN_color_palettes.md) |
| 20-29 | SPEC | 技術仕様 | [20_SPEC_quill_integration.md](mdc:docs/20_SPEC_quill_integration.md) |
| 30-39 | API | API仕様 | 30_API_endpoints.md |
| 40-49 | GUIDE | ガイド・手順書 | 40_GUIDE_deployment.md |
| 50-59 | STRATEGY | 戦略・計画 | [50_STRATEGY_implementation.md](mdc:docs/50_STRATEGY_implementation.md) |
| 70-79 | TEST | テスト・品質 | 70_TEST_unit_specs.md |
| 80-89 | DEPLOY | デプロイ・運用 | 80_DEPLOY_production.md |
| 90-99 | MISC | その他・補助資料 | 90_MISC_references.md |

## AIフレンドリーな構造

### サイズ制限ガイドライン

| レイヤー | 推奨サイズ | 最大サイズ | 目的 |
|----------|------------|------------|------|
| SUMMARY | 2-3KB | 5KB | 全体把握・方針決定 |
| DETAIL | 5-7KB | 10KB | 設計・仕様理解 |
| IMPL | 3-5KB | 8KB | 実装時の詳細参照 |

### 必須テンプレート要素

すべてのドキュメントには以下を含める：

```markdown
# [タイトル]

**カテゴリ**: [CATEGORY] | **レイヤー**: [SUMMARY/DETAIL/IMPL] | **更新**: YYYY-MM-DD  
**担当**: [担当者名] | **依存**: [関連ファイル名] | **タグ**: #frontend #backend #ai

## 🎯 TL;DR（30秒で読める要約）

- **目的**: [何のためのドキュメントか]
- **対象**: [誰が読むべきか]  
- **成果物**: [このドキュメントで得られるもの]
- **次のアクション**: [読了後にすべきこと]

## 🔗 関連ドキュメント

| 種別 | ファイル名 | 関係性 |
|------|-----------|--------|
| 依存 | [XX_CATEGORY_name.md] | [このドキュメントが前提とする] |
| 関連 | [XX_CATEGORY_name.md] | [参照すると理解が深まる] |

## 📊 メタデータ

- **複雑度**: Low/Medium/High
- **推定読了時間**: [X分]
- **更新頻度**: [高/中/低]
```

## 分割戦略

### 大型ドキュメント（10KB超）は分割する

**機能分割パターン**:
```
20_SPEC_large_feature.md →
├── 20_SPEC_feature_summary.md (概要)
├── 21_SPEC_feature_setup.md (セットアップ)
├── 22_SPEC_feature_details.md (詳細仕様)
└── 23_SPEC_feature_impl.md (実装)
```

## 自動化ツール

### 新規ドキュメント作成
```bash
./docs/scripts/new_doc.sh CATEGORY title
```

### ドキュメント分析
```bash
./docs/scripts/analyze_docs.sh
```

### 一覧生成
```bash
./docs/scripts/list_docs.sh > docs/INDEX.md
```

## ファイル管理

### アーカイブ処理
- 不要ファイルは `docs/Archive/old-docs/` に移動
- 削除理由をREADMEに記載

### 更新時の注意
- ファイル名は変更しない（履歴保持）
- 大幅変更時は新バージョン作成
- 関連ファイルからの参照を更新

## 品質チェックポイント

1. **TL;DRセクション**: 30秒で読める要約があるか
2. **関連ドキュメント表**: 依存関係が明記されているか
3. **サイズ制限**: 推奨サイズ内に収まっているか
4. **メタデータ**: 複雑度・読了時間が記載されているか
5. **循環参照**: A→B→A の関係がないか

これらのルールに従うことで、AIアシスタントが効率的にドキュメントを処理でき、コンテキスト超過を防げます。

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/kamekamek)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/kamekamek)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
