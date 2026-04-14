## jamstack-blog

> - `src/app/` … Next.jsのApp Router配下。ページやグローバルCSSなどを配置

# プロジェクト開発ルール

## ディレクトリ構成
- `src/app/` … Next.jsのApp Router配下。ページやグローバルCSSなどを配置
- `src/components/` … 再利用可能なUIコンポーネント
- `src/libs/` … APIクライアントやユーティリティ
- `public/` … 静的ファイル
- `openapi.yaml` … APIスキーマ定義（orvalで型生成）

## コーディング規約
- **TypeScript**を必須とし、型安全を重視する
- APIレスポンスの型は`openapi.yaml`からorvalで自動生成
- コンポーネントは原則`function component`で記述
- 再利用可能なUIは`src/components/`配下に配置
- ページ固有のロジックは`src/app/`配下に記述

## スタイル・デザイン
- **Tailwind CSS**を利用し、ユーティリティクラスでスタイリング
- v4を使用する
- グローバルなカスタムCSSは`src/app/globals.css`に記述
- Shikiで生成されるコードブロック（`pre.shiki`）には`globals.css`でパディング・マージン・角丸・横スクロールを付与
- カスタムクラスや追加スタイルは`globals.css`に追記

## API設計・利用
- APIスキーマは`openapi.yaml`で管理し、変更時はorvalで型を再生成
- APIクライアントは`src/libs/api/generated.ts`などに自動生成
- APIキー等のシークレットは`.env`や環境変数で管理し、コードに直書きしない

## コンポーネント設計
- Presentational/Containerパターンを意識し、UIとロジックを分離
- 目次や日付フォーマットなどの共通処理は専用コンポーネント化
- コードブロックのハイライトはShiki＋カスタムCSSで対応

## 命名規則
- `src/app/`配下は**ルーティングに関わるため必ずkebab-case**で命名する
- `src/components/`配下は**PascalCase**で命名する
- ただし、`src/components/ui/`配下は**shadcn/uiの思想に従いkebab-case**で命名する
- ファイル名・ディレクトリ名は上記ルールに従うこと
- 型・インターフェース名は`PascalCase`
- 変数・関数名は`camelCase`

## 自動生成ファイルの扱い
- `src/libs/api/generated.ts`は**自動生成ファイルのため、手動で変更を加えてはいけない**

## その他
- コミットメッセージは簡潔かつ内容が分かるように記述
- 不要なファイルやコードはこまめに削除
- `.env`やAPIキーなどの機密情報はGit管理しない

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/hoshico)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/hoshico)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
