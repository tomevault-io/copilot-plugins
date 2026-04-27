## minimallauncherapp

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## プロジェクト概要

MinimalLauncherApp は Android 向けのミニマリストランチャーアプリケーションです。Jetpack Compose と従来の View システムを組み合わせたハイブリッド構成で、Hilt による依存性注入を採用しています。

## 開発コマンド

### ビルド

```bash
# アプリのビルド
./gradlew build

# デバッグビルド
./gradlew assembleDebug

# リリースビルド
./gradlew assembleRelease

# クリーンビルド
./gradlew clean build
```

### テスト実行

```bash
# ユニットテスト実行
./gradlew test

# テストレポート生成付きでテスト実行
./gradlew test jacocoTestReport

# 特定のテストのみ実行
./gradlew test --tests "com.kireaji.minimallauncherapp.*"
```

### その他の開発タスク

```bash
# 依存関係の確認
./gradlew dependencies

# Firebase Crashlytics マッピングファイル ID 注入
./gradlew injectCrashlyticsMappingFileIdDebug
./gradlew injectCrashlyticsMappingFileIdRelease
```

## アーキテクチャ構成

### レイヤー構造

```
app/src/main/java/com/kireaji/minimallauncherapp/
├── ui/                  # プレゼンテーション層
│   ├── compose/         # Jetpack Compose 画面
│   ├── viewmodel/       # ViewModel クラス
│   └── Fragment/Activity # 従来の View システム
├── usecase/             # ビジネスロジック層
├── data/                # データ層
│   └── model/           # データモデル
└── di/                  # Hilt 依存性注入設定
```

### 主要コンポーネント

- **FullscreenActivity**: メインアクティビティ。ViewPager2 でカレンダーとアプリリストを切り替え
- **AppListFragment/AppListScreen**: インストール済みアプリ一覧表示（Compose）。インデックスバーによる高速ナビゲーション対応
- **CalendarFragment/CalendarScreen**: カレンダー表示（Compose）
- **AppUseCase**: アプリケーション情報の取得・管理ロジック
- **Hilt DI**: `@HiltAndroidApp`、`@AndroidEntryPoint`、`@Module` によるモジュール構成

### 技術スタック

- **UI**: Jetpack Compose + 従来の View システム（ViewPager2）
- **DI**: Hilt 2.44
- **ビルドツール**: Gradle 8.10.1、Kotlin 1.8.20
- **最小 SDK**: 26 / ターゲット SDK: 36
- **Java バージョン**: 17
- **テスト**: JUnit 4.13.2、Mockito 3.12.4
- **Firebase**: Analytics、Crashlytics、Performance Monitoring

### CI/CD

GitHub Actions による自動ビルド・テスト（`.github/workflows/android_ci.yml`）：
- develop ブランチへの push/PR で起動
- Java 17 環境でビルド・テスト実行
- JaCoCo によるカバレッジレポート生成（最小カバレッジ: 全体 40%、変更ファイル 60%）

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/takumi-saito) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-11 -->
