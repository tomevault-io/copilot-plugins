## badwave-reactnative

> このドキュメントは、`badwave-mobile` プロジェクトの概要、アーキテクチャ、および開発プラクティスを、将来の対話のための指示コンテキストとして提供します。

# GEMINI.md: badwave-mobile プロジェクトガイド

このドキュメントは、`badwave-mobile` プロジェクトの概要、アーキテクチャ、および開発プラクティスを、将来の対話のための指示コンテキストとして提供します。

## 1. プロジェクト概要

`badwave-mobile` は、React Native と Expo で構築されたモバイル音楽ストリーミングアプリケーションです。ユーザーは曲を再生したり、プレイリストを作成したり、音楽を閲覧したりできます。このアプリケーションは、データベースや認証を含むバックエンドサービスに Supabase を使用しています。

### 主要技術

- **フレームワーク**: [React Native](https://reactnative.dev/) と [Expo](https://expo.dev/)
- **言語**: [TypeScript](https://www.typescriptlang.org/) (厳格モード)
- **バックエンド**: [Supabase](https://supabase.io/) (データベース、認証)
- **ローカル DB**: [Drizzle ORM](https://orm.drizzle.team/) + SQLite (expo-sqlite)
- **ナビゲーション**: [Expo Router](https://docs.expo.dev/router/introduction/) (ファイルベースルーティング)
- **データフェッチ**: データフェッチ、キャッシュ、永続化のための [TanStack Query](https://tanstack.com/query/latest)。
- **状態管理**: グローバルな状態管理 (例: プレイヤー UI の状態) のための [Zustand](https://github.com/pmndrs/zustand)。
- **オーディオ再生**: バックグラウンドオーディオのための [React Native Track Player](https://react-native-track-player.js.org/)。
- **ローカルストレージ**: 高速なキーバリューストレージのための [React Native MMKV](https://github.com/mrousavy/react-native-mmkv) と、Supabase セッション永続化のための `AsyncStorage`。
- **テスト**: [Jest](https://jestjs.io/) と [React Testing Library](https://testing-library.com/docs/react-native-testing-library/intro/)。

### アーキテクチャ

このプロジェクトは、**ローカルファースト**アーキテクチャを採用しています。UI はローカル SQLite から読み込み、バックグラウンドで Supabase と同期します。

#### ファイル階層図

```
badwave-mobile/
├── app/              # Expo Routerによる画面とルート
│   ├── (tabs)/       # タブナビゲーション用のレイアウトと画面
│   │   ├── _layout.tsx
│   │   ├── index.tsx   # ホーム画面
│   │   ├── library.tsx
│   │   ├── search.tsx
│   │   └── spotlights.tsx     # Spotlightsタブのメイン画面
│   ├── playlist/     # プレイリスト詳細画面など
│   └── _layout.tsx     # ルートレイアウト
├── actions/          # Supabaseとのデータ通信処理
│   ├── getSongs.ts
│   ├── createPlaylist.ts
│   └── ...
├── components/       # 再利用可能なUIコンポーネント
│   ├── common/
│   ├── player/
│   ├── item/         # SongItem, ListItem, PlaylistItem など
│   └── spotlights/        # Spotlights関連コンポーネント
├── constants/        # 定数
│   ├── index.ts      # キャッシュキー、設定など
│   └── errorMessages.ts  # エラーメッセージ定数
├── hooks/            # ビジネスロジックと状態管理
│   ├── data/         # ローカルDB読み込み用フック (useGetLocalSongs など)
│   ├── sync/         # Supabase → ローカルDB同期フック
│   ├── mutations/    # データ変更用フック (useLikeMutation など)
│   ├── stores/       # Zustand ストア
│   └── ...
├── lib/              # ユーティリティ
│   ├── supabase.ts   # Supabaseクライアント
│   ├── db/           # ローカルSQLite設定・スキーマ
│   └── utils/        # ユーティリティ関数
│       └── retry.ts  # リトライ機能
├── services/         # バックグラウンドサービス
│   └── PlayerService.ts
├── providers/        # React Context プロバイダー
│   ├── AuthProvider.tsx
│   └── SyncProvider.tsx  # バックグラウンド同期
└── __tests__/        # Jestによるテスト (ソース構造をミラーリング)
    └── ...
```

#### 各ディレクトリの役割

- **`app/`**: Expo Router によって管理されるすべての画面とルートが含まれます。
- **`actions/`**: Supabase バックエンドと対話する非同期関数を保持します。
- **`components/`**: 再利用可能な React コンポーネント。すべてのリストアイテムは `React.memo` で最適化されています。
- **`hooks/`**: ビジネスロジックをカプセル化するカスタムフック。
  - `data/`: ローカル SQLite からデータを読み込むフック
  - `sync/`: Supabase からローカルへの同期フック
  - `mutations/`: データ変更用フック（楽観的更新対応）
- **`lib/`**: サードパーティサービスの初期化と共有ユーティリティ関数。
- **`providers/`**: React コンテキストプロバイダー。
- **`services/`**: バックグラウンドサービス。

## 2. データフロー（ローカルファースト）

```
[読み込み] UI ← useGetLocal* ← SQLite (即座に表示)
[同期]    Supabase → useSync* → SQLite (バックグラウンド)
[書き込み] UI → useMutation → Supabase → SQLite
```

### ミューテーションの特徴

1. **楽観的更新**: `onMutate`で即座に UI を更新、失敗時はロールバック
2. **リトライ機能**: `withSupabaseRetry`でネットワークエラー時に自動リトライ
3. **エラーメッセージ**: `constants/errorMessages.ts`で一元管理

### 同期の特徴

1. **トランザクション**: SQLite トランザクションでデータ整合性を保証
2. **差分同期**: Upsert + 差分削除（完全削除ではない）
3. **安全なロールバック**: 失敗時は自動ロールバック

## 3. はじめに

### 前提条件

- Node.js (LTS バージョン)
- npm または yarn
- Expo CLI
- Supabase の URL とキーを含む環境ファイル。

### ビルドと実行

1. **依存関係のインストール**:

   ```bash
   npm install
   ```

2. **アプリケーションの実行**:

   ```bash
   npx expo start
   ```

3. **特定のプラットフォームでの実行**:

   ```bash
   npm run android
   npm run ios
   ```

### テスト

```bash
npm test
```

## 4. 開発規約

- **スタイル**: `StyleSheet.create()` を使用。
- **コンポーネント最適化**: リストアイテムは `React.memo` + カスタム比較関数を使用。
- **TypeScript**: 厳密な型付け。`any`の使用は避ける。
- **エラーハンドリング**: `constants/errorMessages.ts`の定数を使用。
- **リトライ**: ネットワーク依存の操作には`withSupabaseRetry`を使用。
- **楽観的更新**: ユーザー操作に対する即座のフィードバックを実現。
- **TDD**: テスト駆動開発を推奨。

### コミット規約

Conventional Commits の標準に従ってください。

```
feat: 新機能
fix: バグ修正
refactor: リファクタリング
test: テスト追加
docs: ドキュメント更新
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/bads1de)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/bads1de)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
