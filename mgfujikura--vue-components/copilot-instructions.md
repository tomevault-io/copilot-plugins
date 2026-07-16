## project-conventions

> プロジェクト全体の開発哲学・コーディング規約・コミュニケーション方針


# プロジェクト概要

- Vue 3 + TypeScript + Vite 構成の Vue 3 コンポーネントライブラリ（`@mgfujikura/vue-components`）。
- ディレクトリ構成や命名規則は既存の `src/` 以下に従う。
  - `src/components/*.vue`（PascalCase）
  - `src/composable/*.ts`

# 開発哲学

- 責務分離を徹底し、UI とロジックを明確に分離する。
- Composable API を優先し、DOM 管理・Observer 管理・イベント管理は **必ず composable 内に閉じ込める**。
- UI コンポーネントは **最小限のマークアップ** で動作するように設計する。
- 命名は厳密に行い、拡張性・意味的正確性・構造的整合性を重視する。
- 推測で補完せず、要件が曖昧な場合は質問する。

# コーディング規約

- TypeScript を優先する。
- Composition API（`<script setup>`）を推奨する。
- 型安全を重視し、`any` は避ける。型定義が複雑で `any` を使う場合は事前に相談する。
- 既存の ESLint / Prettier 設定とコードスタイルに従う。

# 命名規則

- 名前は「構造」「意味」「拡張性」を基準に選ぶ。
- 新しいコンポーネント名を提案する際は、**代替案を複数提示し、意味的差分を説明する**。

# コード生成方針

- 曖昧な指示には、コード生成の前に質問する。特に以下の場合は必ず確認する：
  - 要件が曖昧なとき / 命名の意図が不明なとき
  - レイアウト要件が不足しているとき / composable の責務範囲が判断できないとき
- 生成時は「最小構成で動く」ことを優先し、過剰な抽象化は避ける。
- 目的・選択したアプローチの理由・拡張性/保守性の観点を簡潔に添える。

# トーンとスタイル

- 過度にカジュアルにせず、技術的に正確かつ簡潔に説明する。
- 推測ではなく、根拠に基づいた提案を行う。

---
> Source: [mgfujikura/vue-components](https://github.com/mgfujikura/vue-components) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-16 -->
