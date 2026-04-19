## fbt-v2

> - shadcn/ui をベースとしたコンポーネントの使用

---
description:
globs: *.tsx
alwaysApply: false
 

# UI/UX 設計・実装ルール

## 1. デザインシステム

### 重要度: 最高

- shadcn/ui をベースとしたコンポーネントの使用
- **既存の UI は承認なしでの変更を禁止**
- コンポーネントのカスタマイズは最小限に抑える

```typescript
// ✅ 良い例：shadcn/uiコンポーネントをそのまま使用
import { Button } from "@/components/ui/button";

// ❌ 悪い例：不必要なカスタマイズ
const CustomButton = styled(Button)`
  // 独自のスタイリング
`;
```

## 2. スタイリング規約

### 重要度: 高

### Tailwind CSS の使用

- ユーティリティクラスを優先的に使用
- カスタムクラスは`@layer`で定義
- 命名規則は`kebab-case`を使用

```typescript
// ✅ 良い例
<div className="flex items-center justify-between p-4">

// ❌ 悪い例
<div style={{ display: 'flex', alignItems: 'center' }}>
```

## 3. レスポンシブデザイン

### 重要度: 高

- モバイルファーストアプローチ
- Tailwind のブレークポイントを使用
  - sm: 640px
  - md: 768px
  - lg: 1024px
  - xl: 1280px
  - 2xl: 1536px

## 4. アクセシビリティ

### 重要度: 高

- WAI-ARIA ガイドラインの遵守
- キーボード操作のサポート
- 適切なコントラスト比の確保
- スクリーンリーダー対応

## 5. アニメーションとトランジション

### 重要度: 中

- 過度なアニメーションを避け、必要最小限に留める
- 利用ライブラリの選定や導入は PR で合意を取る

## 6. フォーム設計

### 重要度: 高

- shadcn/ui のフォームコンポーネントを使用
- バリデーションメッセージは明確に表示
- 入力補助の実装（オートコンプリートなど）

## 7. 重要な制約事項

### 重要度: 最高

1. UI 変更の制限

- 変更が必要な場合は `globals.mdc` の承認フローに従うこと
- レイアウト、色、フォント、間隔などの変更は特に注意

2. 承認不要の軽微変更（例外）

- 文言の誤字修正
- アクセシビリティ属性の追加/改善（aria-* 等）
- 明確なバグ修正（意図しない崩れ/動作不良）
- 計測タグの追加（既存ポリシーに準拠）

2. コンポーネントの追加

- 新規コンポーネントは shadcn/ui の設計原則に従う
- 既存のコンポーネントの再利用を優先

## 8. エラー表示とフィードバック

### 重要度: 高

- プロジェクト標準の通知/トースト実装を使用（存在しない場合は PR で追加提案）
- エラーメッセージは具体的で分かりやすく
- ローディング状態の適切な表示

## 9. アイコンとイメージ

### 重要度: 中

- Lucide Icons を標準として使用
- SVG アイコンの最適化
- 画像は`next/image`で最適化

## 10. ダークモード対応

### 重要度: 高

- `dark:`プレフィックスでスタイリング
- システム設定との連動
- コントラスト比の維持

## 11. コンポーネント設計原則

### 重要度: 高

- 単一責任の原則
- Props 経由での柔軟なカスタマイズ
- 適切なコンポーネント分割

```typescript
// ✅ 良い例
interface CardProps {
  title: string;
  children: React.ReactNode;
  className?: string;
}

// ❌ 悪い例
interface CardProps {
  title: string;
  titleColor: string; // 不要なカスタマイズ
  customPadding: string; // 避けるべき
}
```

## 注意事項

1. デザインの一貫性

- コンポーネントライブラリの一貫した使用
- カスタムスタイルの最小化
- デザイントークンの遵守

2. パフォーマンス

- 不要なアニメーションの削除
- 画像の最適化
- バンドルサイズの監視

3. 品質管理

- コンポーネントのストーリーブック作成
- ビジュアルリグレッションテスト
- クロスブラウザテスト

4. ドキュメント

- コンポーネントの使用例
- Props の型定義
- デザインシステムのガイドライン

これらのルールは、プロジェクトの一貫性と保守性を確保するために重要です。
変更が必要な場合は、必ずチームでの承認プロセスを経てください。

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/yasu-dev) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
