## maymai-dev

> | 共有ブラウザスクリプト | kebab-case の `.ts` を優先 | `scroll-header.ts`, `form-validation.ts` |


# スクリプト命名規則

## ファイル命名

| 種別 | 規則 | 例 |
|---|---|---|
| 共有ブラウザスクリプト | kebab-case の `.ts` を優先 | `scroll-header.ts`, `form-validation.ts` |
| 補助ユーティリティ | kebab-case | `focus-trap.ts`, `clamp-text.ts` |
| フレームワーク UI | PascalCase | `Carousel.tsx`, `Accordion.jsx` |

## ディレクトリ構成

```text
src/
├── scripts/
│   ├── dom/
│   │   └── scroll-header.ts
│   └── lib/
│       └── focus-trap.ts
└── components/
    └── Accordion.tsx
```

## 命名規則

| 種別 | 規則 | 例 |
|---|---|---|
| 変数・関数 | camelCase | `userName`, `initAccordion` |
| 定数 | UPPER_SNAKE_CASE | `MAX_RETRY`, `HEADER_OFFSET` |
| クラス | PascalCase | `CarouselController` |

## 運用ルール

- 共有スクリプトは `src/scripts/` に集約する
- そのコンポーネント専用の処理は `.astro` の `<script>` または同名近接ファイルに閉じる
- `src/` 配下から import したスクリプトは Astro / Vite がバンドルする
- `public/` 配下の JS は素通しで配信されるため、最適化や依存解決を期待しない

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ozekimasaki) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-11 -->
