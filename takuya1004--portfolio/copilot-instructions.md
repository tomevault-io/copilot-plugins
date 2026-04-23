## portfolio

> - 簡潔かつ技術的に正確なTypeScriptコードを書く


# TypeScript・React Native・Expo・モバイルUI開発ガイドライン

## コードスタイルと構造
- 簡潔かつ技術的に正確なTypeScriptコードを書く
- 関数型・宣言的なパターンを使い、クラスは使用しない
- 重複を避け、繰り返し処理やモジュール化を優先する
- `isLoading` や `hasError` のような補助動詞付きの説明的な変数名を使用
- ファイル構成: エクスポートコンポーネント / サブコンポーネント / ヘルパー / 静的コンテンツ / 型定義
- Expo公式ドキュメントに従って設定: https://docs.expo.dev/

## 命名規則
- ディレクトリは小文字+ダッシュ（例: `components/auth-wizard`）
- コンポーネントは名前付きエクスポートを推奨

## TypeScriptの使用方針
- すべてのコードでTypeScriptを使用
- `type`よりも`interface`を優先
- `enum`は避け、代わりにマップを使用
- 関数型コンポーネント + interface を使用
- `strict`モードを有効にして型安全性を強化

## 構文とフォーマット
- 純粋関数には `function` キーワードを使用
- 条件分岐での不要な波括弧を避ける
- 宣言的なJSXを使用
- Prettierでコードを自動整形

## UIとスタイリング
- ExpoのUIコンポーネントを使用
- Flexbox + `useWindowDimensions` でレスポンシブ対応
- styled-components または Tailwind CSS を使用
- `useColorScheme` でダークモード対応
- ARIAやネイティブa11yプロップでアクセシビリティを担保
- `react-native-reanimated` と `gesture-handler` で高速アニメーション

## セーフエリアの管理
- `SafeAreaProvider` で全体をラップ
- トップレベルは `SafeAreaView` で囲む
- スクロールには `SafeAreaScrollView` を使用
- padding/marginはハードコードせず、フックやViewで管理

## パフォーマンス最適化
- `useState`・`useEffect`の多用は避け、`Context`や`Reducer`を使う
- App起動には `AppLoading` や `SplashScreen` を使用
- 画像は WebP形式・サイズ指定・expo-imageで遅延読み込み
- Suspense + dynamic importでコード分割
- 内蔵ツールとExpoデバッグでパフォーマンス監視
- `useMemo`・`useCallback`・`memo`で再レンダリング最適化

## ナビゲーション
- `react-navigation` を使用し、スタック/タブ/ドロワーに対応
- Deep LinkingとUniversal Linksを活用
- `expo-router`で動的ルーティングを導入

## 状態管理
- グローバル状態は `React Context + useReducer` で管理
- データ取得は `react-query` でキャッシュ付き取得
- 複雑な状態は `Zustand` や `Redux Toolkit` を検討
- URLパラメータは `expo-linking` で処理

## エラーハンドリングとバリデーション
- 実行時バリデーションに `Zod` を使用
- ログ送信には `Sentry` や同等ツールを活用
- エラー処理の原則:
  - 冒頭でエラー処理
  - 早期return
  - 不要な `else` を避ける
  - グローバルエラーバウンダリを設置
  - `expo-error-reporter` で本番エラー報告

## テスト
- 単体テスト：Jest + React Native Testing Library
- 統合テスト：Detoxで主要フロー確認
- Expoテスト環境を活用
- SnapshotテストでUIの一貫性を担保

## セキュリティ
- ユーザー入力をサニタイズしてXSS防止
- `react-native-encrypted-storage` でデータを暗号化
- HTTPS＋認証で通信を保護
- Expoのセキュリティガイドに従う: https://docs.expo.dev/guides/security/

## 国際化（i18n）
- `react-native-i18n` または `expo-localization` を使用
- 多言語＋RTL対応
- テキストスケーリングとフォント調整でa11y強化

## キーとなる慣習
1. Expoのマネージドワークフローを使用
2. モバイルWeb Vitalsを重視
3. `expo-constants` で環境変数管理
4. `expo-permissions` で権限制御
5. `expo-updates` でOTA配信
6. Expoの配布・公開手順に従う: https://docs.expo.dev/distribution/introduction/
7. iOS/Android両方で徹底検証

## APIドキュメント
- Expo公式ガイド: https://docs.expo.dev/
- また、Context7 MCPを使用することでドキュメントを理解することも可能
- 各種ViewやBlueprints、拡張仕様はドキュメントを参照

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/takuya1004) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
