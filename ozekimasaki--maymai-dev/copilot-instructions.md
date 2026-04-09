## maymai-dev

> 命名規約の競合解消と採用基準


# 命名規約 競合解消ルール

## 優先順位

| 優先度 | 規約ソース | 採用範囲 |
|---|---|---|
| 1 | プロジェクト既存規約（`AGENTS.md`, `.cursor/rules/*.mdc`） | 全面採用 |
| 2 | Astro の構成原則 | ルーティング、コンポーネント分割、hydrate 方針 |
| 3 | コードレビュー観点 | 保守性、可読性、パフォーマンス |

## 採用する命名

### Astro ファイル

| 対象 | 採用ルール | 例 |
|---|---|---|
| レイアウト | PascalCase + `Layout.astro` | `BaseLayout.astro` |
| コンポーネント | PascalCase | `Header.astro`, `HeroSection.astro` |
| ページ | ルート準拠の kebab-case | `about.astro`, `blog/[slug].astro` |
| 共有スクリプト | kebab-case | `scroll-header.ts` |

### クラス命名

| 観点 | 規約 | 例 |
|---|---|---|
| Block | PascalCase | `.Header`, `.FooterEntry` |
| Element | camelCase | `.Header__navLink`, `.Footer__btnText` |
| Modifier | kebab-case | `.Header__btn--filled` |
| JS 用クラス | `js-` プレフィックス | `.js-header-burger` |
| ユーティリティ | kebab-case | `.pc-only`, `.sp-only`, `.u-anime` |
| 状態クラス | `is-` プレフィックス | `.is-active`, `.is-visible` |

## JS フック

| 方式 | 判定 | 理由 |
|---|---|---|
| `js-` プレフィックスクラス | 採用 | スタイルと責務を分離できる |
| `data-*` 属性での選択 | 補助的に許容 | 状態や値保持が主目的の場合のみ |
| スタイルクラス直接参照 | 禁止 | CSS 変更で JS が壊れやすい |

## CSS 変数

| パターン | 判定 | 例 |
|---|---|---|
| `--color-*` の小文字ケバブ | 採用 | `--color-primary` |
| PascalCase 変数名 | 禁止 | `--ColorPrimary` は使わない |

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ozekimasaki)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/ozekimasaki)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
