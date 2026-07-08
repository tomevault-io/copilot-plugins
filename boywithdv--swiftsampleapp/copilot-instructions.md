## swiftsampleapp

> 位置情報を使ったSNS iOS アプリ「LocaSocial」。Flutter版 `location_social_media` と同じFirebaseプロジェクト (`locationbasedinformationshare`) を共有。

# CLAUDE.md — SwiftSampleApp (LocaSocial)

## プロジェクト概要

位置情報を使ったSNS iOS アプリ「LocaSocial」。Flutter版 `location_social_media` と同じFirebaseプロジェクト (`locationbasedinformationshare`) を共有。
UIKit + SwiftUI ハイブリッド、RxFlow/RxSwift アーキテクチャ。

## ビルド・実行コマンド

```bash
open SwiftSampleApp.xcodeproj
xcodebuild -scheme SwiftSampleApp -destination 'platform=iOS Simulator,name=iPhone 16'
```

## 依存パッケージ（Xcode SPM で管理）

- **RxFlow** 2.13.2
- **RxSwift** 6.10.1 (RxCocoa含む)
- **FirebaseCore / FirebaseAuth / FirebaseFirestore** (firebase-ios-sdk)
- **FirebaseAnalytics / FirebaseRemoteConfig / FirebaseCrashlytics**

## Firebase セットアップ（手動ステップ）

1. Xcode > File > Add Package Dependencies: `https://github.com/firebase/firebase-ios-sdk`
2. Firebase Console (`locationbasedinformationshare`) から iOS バンドル ID で `GoogleService-Info.plist` を発行
3. `SwiftSampleApp/` ターゲットに追加

## Info.plist に必要な追加キー

- `NSLocationWhenInUseUsageDescription` — マップ機能に必要

## アーキテクチャ

### 遷移フロー

```
SceneDelegate → AppFlow → SplashFlow → (authRequired) → AuthFlow → LoginVC/RegisterVC
                                      → (tabBarIsRequired) → TabFlow
                                                              ├── TimelineFlow (Tab 0)
                                                              ├── SwiperFlow   (Tab 1)
                                                              ├── MapFlow      (Tab 2)
                                                              ├── SearchFlow   (Tab 3)
                                                              └── ProfileFlow  (Tab 4)
                                                                   └── ChatFlow (sub-flow)
```

### レイヤー構成

```
DesignSystem/    — AppTheme, AppTabBarAppearance, UIColor+Hex
Models/          — UserModel, UserPost, Message, Comment (Codable)
Services/        — AuthService, FirestoreService, UserRepository, PostRepository,
                   MessageRepository, CommentRepository, RemoteConfigService, LocationService
Components/      — AvatarImageView, SNSCardView, PrimaryButton
Flows/           — AppFlow, AuthFlow, TabFlow, TimelineFlow, SwiperFlow, MapFlow,
                   SearchFlow, ProfileFlow, ChatFlow, SplashFlow
Features/
  Auth/          — Login/Register VC + VM
  Timeline/      — Timeline + PostCell + CreatePost
  Swiper/        — SwiftUI card swiper
  Map/           — MKMapView + UserAnnotation
  Search/        — UISearchController + UserSearchCell
  Profile/       — Profile + EditProfile
  PostDetail/    — SwiftUI post + comments
  UserProfile/   — Other user view
  FollowList/    — Followers/Following list
  Chat/          — AllChats + ChatThread (SwiftUI)
  Splash/        — Splash + auth routing
```

### デザインシステム

| Token | Light | Dark |
|---|---|---|
| Primary | `#01896C` | `#37B6E9` |
| Secondary | `#F45479` | `#8BF8C4` |
| Background | `#F2F2F7` | `#192734` |
| Surface | `#FFFFFF` | `#22303C` |

### SwiftUI HostingVC パターン（Swiper / PostDetail / Chat）

`BaseViewModel + ObservableObject` 双方に準拠。UIKit Stepper は `SwiperHostingViewController` 等が担い、SwiftUI側は `@ObservedObject` で viewModel を参照。

## Xcode に手動追加が必要なファイル

新規ファイルは以下の順序でターゲット追加すること（`.pbxproj` は直接編集禁止）:

1. DesignSystem/
2. Models/
3. Services/
4. Components/
5. Flows/（AuthFlow, ChatFlow, TimelineFlow, SwiperFlow, MapFlow, SearchFlow, ProfileFlow）
6. Features/ 各フォルダ

## 不要になったファイル（削除推奨）

以下のファイルは SNS 化で置き換わったが、ファイルシステム上に残存している:

- `Flows/HomeFlow.swift`, `BrowsingFlow.swift`, `ReservationFlow.swift`, `FavoriteFlow.swift`, `MyPageFlow.swift`
- `Features/HomePage/` 配下（ラーニング画面群）
- `Features/BrowsingHistory/`, `Features/Reservation/`, `Features/Favorite/`, `Features/MyPage/`
- `Features/RootTabBarViewController.swift`, `Features/RootViewController.swift`

Xcode の Navigator からターゲット外しただけでも OK（ビルドエラーにならない）。

## コーディング規約

- UIはすべてコードベース（Storyboard不使用）
- ViewControllerは `// MARK: -` セクションで整理
- `[weak self]` + `DisposeBag` で RxSwift メモリ管理
- SwiftUI HostingVC パターン: `BaseViewModel` + `ObservableObject` 双方に準拠
- フォントは端末の文字サイズ設定（Dynamic Type）の影響を受けない。`SceneDelegate` で `window.traitOverrides.preferredContentSizeCategory = .large` を掛けてアプリ全体を標準サイズに固定しているため。端末設定に追従させたい場合はこの1行を外す

---
> Source: [boywithdv/SwiftSampleApp](https://github.com/boywithdv/SwiftSampleApp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
