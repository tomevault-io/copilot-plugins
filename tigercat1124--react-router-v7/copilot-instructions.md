## react-router-v7

> - **CSSファイル（.css）の作成・使用は厳格に禁止されています**

# プロジェクトルール

## ⚠️ 重要: スタイリング規則

### CSSファイルの使用禁止（厳守）
- **CSSファイル（.css）の作成・使用は厳格に禁止されています**
- 新しい `.css` ファイルを作成しないでください
- 既存のCSSファイルを編集しないでください（例外を除く）
- インラインスタイル（`style` 属性）の使用も避けてください

### スタイリングの実装方法（必須）
- **すべてのスタイリングは Tailwind CSS のユーティリティクラスを使用してください**
- コンポーネント内で `className` プロパティにTailwindクラスを指定
- 例: `className="bg-[#0066CC] text-white px-8 py-3 rounded font-semibold hover:bg-[#0052A3]"`
- カスタムスタイルが必要な場合は、Tailwindの設定ファイルで拡張するか、任意値構文（`bg-[#色コード]`）を使用

### 例外（限定的）
- `app/app.css`: `@import "tailwindcss"` のみを含める（編集不可）
- Storybookの `preview.ts` で `app.css` をインポートする必要がある
- `stories/` ディレクトリ内の既存のサンプルCSSファイルは無視（使用しない）

### コード生成時の注意
- コンポーネントを作成する際は、必ずTailwind CSSクラスを使用してください
- CSSファイルやインラインスタイルを提案しないでください
- スタイリングが必要な場合は、Tailwindのユーティリティクラスを提案してください

## コンポーネント開発
- TypeScriptを使用
- React関数コンポーネントを使用
- Propsは型定義を必ず行う
- 再利用可能なコンポーネントは `app/components/` に配置

## コードスタイル
- ESLint/Prettierの設定に従う
- コンポーネント名はPascalCase
- ファイル名はコンポーネント名と一致させる

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/tigercat1124) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
