## golf-score-manage

> このファイルは、このリポジトリでコードを扱う際のClaude Code (claude.ai/code) へのガイダンスを提供します。

# CLAUDE.md

このファイルは、このリポジトリでコードを扱う際のClaude Code (claude.ai/code) へのガイダンスを提供します。

## プロジェクト概要

ゴルフのスコア履歴を管理するWebアプリケーションで、ユーザーがゴルフスコアの記録を追跡・管理できるように設計されています。

**詳細な設計については以下のドキュメントを参照してください：**
- **基本仕様**: `.claude/BasicSpec.md`
- **フロントエンド設計**: `.claude/FrontendDesign.md`
- **バックエンドAPI設計**: `.claude/BackendDesign.md`
- **データベース設計**: `.claude/DatabaseDesign.md`

## 技術スタック

プロジェクトは以下の技術で構築されます：
- **フロントエンド**: React (TypeScript) + Material UI
- **バックエンド**: Node.js (Express or Firebase Functions)
- **データベース**: Google Firestore
- **認証**: Firebase Authentication (Email/Password, OAuth)
- **デプロイ**: Firebase Hosting + Firebase Functions

## 主な機能

1. **ユーザ管理** - 会員登録/ログイン/ログアウト、プロフィール編集
2. **スコア入力** - 新規ラウンドの追加、同伴者選択
3. **スコア履歴表示** - ラウンド一覧・詳細表示
4. **統計・分析** - 期間指定による平均スコア、コース別平均
5. **同伴者情報** - 同伴者のプロフィールリンク
6. **管理機能** - ゴルフ場マスタ管理（オプション）

## 現在の状況

このリポジトリは初期セットアップ段階にあり、基本的な構成のみが含まれています：
- 日本語でのプロジェクト説明を含むREADME.md
- 基本設計書（.claude/BasicSpec.md）
- LICENSEファイル
- Node.jsプロジェクト用に設定された.gitignore
- Node.jsとPythonをサポートする.devcontainerセットアップ
- GitHub Dependabotの設定

## 未実装部分

プロジェクトには以下が不足しています：
- package.jsonと依存関係管理
- Firebase設定
- ソースコードのディレクトリとファイル
- ビルド設定
- テストセットアップ
- 実際のアプリケーション実装

## 開発環境

プロジェクトは以下で設定されています：
- **Devcontainer**: Node.jsとPythonの両方の開発環境をサポート
- **Git**: 包括的なNode.js用.gitignoreでリポジトリを初期化
- **GitHub**: devcontainer更新用のDependabotを設定

## データモデル

Firestoreコレクション構成：
- **users** - ユーザー情報（name, email, handicap等）
- **courses** - ゴルフ場情報（name, address, holesCount等）
- **rounds** - ラウンド記録（userId, courseId, playDate, scores等）
- **stats** - 統計キャッシュ（任意）

## 実装時の注意事項

- React + TypeScript + Material UIでモダンなUIを構築
- Firebase Authentication/Firestoreを適切に統合
- レスポンシブデザインを考慮
- 日本語ユーザー向けのUX/UI設計
- データセキュリティとプライバシーを考慮

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/terapyon)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/terapyon)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
