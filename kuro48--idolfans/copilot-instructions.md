## idolfans

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## プロジェクト概要

**イコつな** - =LOVE、≠ME、≒JOY ファン専用のiOS SNSアプリケーション

- **言語**: Swift
- **フレームワーク**: SwiftUI
- **最小対応OS**: iOS 16.0以降
- **バックエンド**: Firebase (Authentication, Firestore, Cloud Storage, Cloud Functions)
- **パッケージマネージャー**: Swift Package Manager (SPM)

### コンセプト
ファン同士の交流を主体としたSNSアプリ。現場レポート、スケジュール管理、グッズ交流を通じて、イコラブファミリーファンが安心して繋がれるコミュニティを提供します。

## 開発環境

### 必須ツール
- Xcode 15以降
- Apple Developer Program アカウント
- Firebase プロジェクト

### 環境セットアップ
```bash
# Firebase CLI インストール
npm install -g firebase-tools

# Firebase ログイン
firebase login

# Firebase プロジェクト初期化
firebase init
```

## アーキテクチャ

### プロジェクト構造
```
IdolFans/
├── App/
│   ├── IdolFansApp.swift           # アプリエントリーポイント
│   └── AppDelegate.swift           # アプリデリゲート
├── Models/
│   ├── User.swift                  # ユーザーモデル
│   ├── Post.swift                  # 投稿モデル
│   ├── Group.swift                 # グループモデル (=LOVE, ≠ME, ≒JOY)
│   ├── Member.swift                # メンバーモデル
│   ├── Event.swift                 # イベントモデル
│   └── Comment.swift               # コメントモデル
├── ViewModels/
│   ├── AuthViewModel.swift         # 認証ロジック
│   ├── HomeViewModel.swift         # タイムラインロジック
│   ├── PostViewModel.swift         # 投稿ロジック
│   ├── ProfileViewModel.swift      # プロフィールロジック
│   └── ScheduleViewModel.swift     # スケジュールロジック
├── Views/
│   ├── Auth/                       # 認証画面
│   ├── Home/                       # ホーム・タイムライン画面
│   ├── Schedule/                   # スケジュール画面
│   ├── Post/                       # 投稿作成画面
│   ├── Profile/                    # プロフィール画面
│   └── Components/                 # 再利用可能コンポーネント
├── Services/
│   ├── AuthService.swift           # Firebase Authentication
│   ├── FirestoreService.swift      # Firestore データ操作
│   ├── StorageService.swift        # Cloud Storage (画像アップロード)
│   └── NotificationService.swift   # プッシュ通知
├── Utils/
│   ├── Extensions.swift            # 拡張機能
│   ├── Constants.swift             # 定数定義
│   └── ImageCompressor.swift       # 画像圧縮
└── Resources/
    ├── Assets.xcassets            # アセット
    ├── GoogleService-Info.plist   # Firebase設定
    └── Localizable.strings        # ローカライゼーション
```

### MVVM パターン
- **Model**: Firestore のデータ構造を反映した Swift 構造体
- **View**: SwiftUI ビュー (宣言的UI)
- **ViewModel**: ビジネスロジックと状態管理 (`@Published`, `ObservableObject`)

## Firestore データ構造

### 主要コレクション
- `/groups/{groupId}` - グループ情報 (=LOVE, ≠ME, ≒JOY)
- `/members/{memberId}` - メンバー情報
- `/users/{userId}` - ユーザープロフィール
- `/posts/{postId}` - 投稿 (text, image, live_report, goods, question)
- `/events/{eventId}` - イベント・スケジュール
- `/likes/{likeId}` - いいね
- `/comments/{commentId}` - コメント
- `/bookmarks/{bookmarkId}` - ブックマーク
- `/attendees/{attendeeId}` - イベント参加予定

### 投稿タイプ
1. **text** - 雑談・日常
2. **image** - 画像投稿
3. **live_report** - 現場レポート (会場、日時、セトリ、感想)
4. **goods** - グッズ紹介
5. **question** - 質問・相談

## 主要機能

### MVP フェーズ (Phase 0)
1. **認証**: メール・パスワード登録、ログイン、パスワードリセット
2. **プロフィール**: ニックネーム、アイコン、推しグループ・メンバー選択
3. **タイムライン**: 投稿一覧、グループフィルター、無限スクロール
4. **投稿**: テキスト・画像投稿 (最大4枚)、現場レポートテンプレート
5. **インタラクション**: いいね、コメント、ブックマーク
6. **スケジュール**: イベントカレンダー、参加予定機能
7. **検索**: ハッシュタグ検索、ユーザー検索
8. **安全機能**: 通報、ブロック

### Phase 1 以降
- フォロー機能
- 通知機能
- DM (ダイレクトメッセージ)
- グッズ交換掲示板

## コーディング規約

### ファイル命名規則
- View: `~View.swift` (例: `HomeView.swift`)
- ViewModel: `~ViewModel.swift` (例: `HomeViewModel.swift`)
- Model: `~.swift` (例: `User.swift`)
- Service: `~Service.swift` (例: `AuthService.swift`)

### 変数・関数命名
- **変数**: lowerCamelCase (例: `userName`, `isLoggedIn`)
- **Bool変数**: `is~`, `has~`, `can~` で始める
- **関数**: 動詞で始める (例: `fetchPosts()`, `uploadImage()`)
- **定数**: UPPER_SNAKE_CASE (例: `MAX_IMAGE_COUNT = 4`)

### コメント規約
```swift
// MARK: - セクション区切り
// TODO: 未実装機能
// FIXME: 修正が必要な箇所
```

## Git ブランチ戦略

```
main        : 本番リリース用 (App Store)
develop     : 開発用メインブランチ
feature/*   : 機能開発 (例: feature/auth-implementation)
bugfix/*    : バグ修正 (例: bugfix/image-upload-crash)
release/*   : リリース準備 (例: release/v1.0.0)
```

### ブランチルール
- `main` には直接コミットしない
- `feature/*` から `develop` へマージ
- リリース時に `develop` から `release/*` を切り、テスト後 `main` へマージ

## 開発タスク

### テスト実行
```bash
# 単体テスト実行
⌘ + U (Xcode)

# UI テスト実行
⌘ + U (Xcode) または個別テストクラスから実行
```

### ビルド・実行
```bash
# シミュレータで実行
⌘ + R (Xcode)

# 実機で実行
Developer アカウント設定後、Xcode から実機を選択して ⌘ + R

# アーカイブ作成 (TestFlight/App Store)
Product > Archive (Xcode メニュー)
```

### 画像処理
- **最大枚数**: 4枚/投稿
- **最大サイズ**: 5MB/枚
- **圧縮**: quality 0.7
- **対応形式**: JPEG, PNG

### Firebase ルール管理
```bash
# Firestore Security Rules デプロイ
firebase deploy --only firestore:rules

# Storage Rules デプロイ
firebase deploy --only storage
```

## セキュリティ考慮事項

### 認証
- パスワード最低8文字
- Firebase Authentication の bcrypt ハッシュ化
- レートリミット自動適用

### データアクセス
- Firestore Security Rules で認証必須
- 自分のデータのみ編集可能
- 投稿・コメント・プロフィールは基本公開 (SNS性質上)

### 画像アップロード
- ファイルサイズ制限: 5MB/枚
- 形式検証: JPEG, PNG のみ
- Storage Rules で認証ユーザーのみアップロード可

## 重要な技術的決定事項

### Swift Package Manager 使用
CocoaPods ではなく SPM で依存関係を管理:
```swift
dependencies: [
    .package(url: "https://github.com/firebase/firebase-ios-sdk", from: "10.0.0"),
    .package(url: "https://github.com/onevcat/Kingfisher", from: "7.0.0")
]
```

### 画像キャッシュ
Kingfisher を使用して画像ダウンロード・キャッシュを効率化

### リアルタイムデータ同期
Firestore の `.addSnapshotListener` でタイムラインをリアルタイム更新

### オフライン対応
Firestore のオフラインキャッシュを有効化

## トラブルシューティング

### よくある問題

**Firebase 接続エラー**
- `GoogleService-Info.plist` が正しく配置されているか確認
- Firebase Console でプロジェクト設定を確認

**画像アップロード失敗**
- Storage Rules が正しく設定されているか確認
- 画像サイズが 5MB 以下か確認
- ネットワーク接続を確認

**ビルドエラー**
- Xcode を最新バージョンに更新
- `Product > Clean Build Folder` を実行
- Derived Data を削除: `~/Library/Developer/Xcode/DerivedData`

## リリース準備

### App Store 申請前チェックリスト
- [ ] すべての機能が動作確認済み
- [ ] 利用規約・プライバシーポリシー実装
- [ ] App Store スクリーンショット準備 (6.5インチ + 5.5インチ)
- [ ] アプリアイコン (1024x1024px) 準備
- [ ] TestFlight βテスト完了
- [ ] バージョン番号設定 (例: 1.0.0)
- [ ] Bundle ID 設定
- [ ] カテゴリ: ソーシャルネットワーキング
- [ ] 年齢制限: 12+

## 参考リンク

### 公式ドキュメント
- [SwiftUI Documentation](https://developer.apple.com/documentation/swiftui)
- [Firebase iOS Documentation](https://firebase.google.com/docs/ios/setup)
- [Human Interface Guidelines](https://developer.apple.com/design/human-interface-guidelines/)

### 学習リソース
- [Hacking with Swift](https://www.hackingwithswift.com/)
- [SwiftUI by Example](https://www.hackingwithswift.com/quick-start/swiftui)
- [Firestore 公式ガイド](https://firebase.google.com/docs/firestore)

## プロジェクト固有の注意事項

### 現場レポート機能
投稿タイプが `live_report` の場合、以下の追加フィールドが必要:
- `venue`: 会場名
- `eventDate`: 日付 (YYYY-MM-DD)
- `setlist`: セットリスト (配列)
- `impressions`: 感想

### グループフィルター
タイムラインは必ず `groupId` でフィルタリング可能にする:
- "すべて" - 全グループ表示
- "=LOVE" - =LOVE のみ
- "≠ME" - ≠ME のみ
- "≒JOY" - ≒JOY のみ

### ハッシュタグ処理
投稿時にテキストから `#` で始まる文字列を自動抽出し、`hashtags` 配列に格納

### 日本語対応
UI は完全日本語対応。ローカライゼーションファイルは `Localizable.strings` に配置

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/kuro48)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/kuro48)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
