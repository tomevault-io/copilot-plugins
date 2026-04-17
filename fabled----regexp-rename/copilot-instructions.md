## regexp-rename

> このプロジェクトは、Rust (Tauri v2) と Vue 3 を使用した高機能な正規表現ファイルリネームツールです。


# 正規表現リネームツール (regexp-rename) 開発ルール

## プロジェクト概要
このプロジェクトは、Rust (Tauri v2) と Vue 3 を使用した高機能な正規表現ファイルリネームツールです。

## 技術スタック
- **Frontend**: Vue 3 (Composition API) + TypeScript + Tailwind CSS
- **Backend**: Rust (Tauri v2)
- **Component Design**: Atomic Design (components/ 内に配置)

## ディレクトリ構成 (Tauri/Vite 標準)
- [src/](cci:9://file:///c:/codes/regexp-rename/src:0:0-0:0): Vue 3 フロントエンド
  - `components/atoms/`, `components/molecules/`, `components/organisms/`: Atomic Design
  - `composables/`: ビジネスロジック
  - `store/`: 状態管理 (Pinia)
- `src-tauri/`: Rust バックエンド

## テスト方針
- フロントエンド: Vitest を使用し、`src/**/__tests__/` に配置（対象コードの近くに置く）
- テストファイル名は `*.spec.ts` を基本とする
- バックエンド: Rust 標準のテスト機能を使い、各モジュール内に記述

## 実装上の注意点
- **設定保存**: 設定（グループ、正規表現、アクティブグループ等）は Rust 側の [save_settings](cci:1://file:///c:/codes/regexp-rename/src-tauri/src/lib.rs:74:4-82:5) / [load_settings](cci:1://file:///c:/codes/regexp-rename/src-tauri/src/lib.rs:59:4-72:5) invoke コマンドを通じて永続化します。
- **IPC通信**: 重い処理やファイル操作は Rust 側 ([execute_rename_files](cci:1://file:///c:/codes/regexp-rename/src-tauri/src/lib.rs:84:4-167:5)) で行い、フロントエンドは表示と入力を担当します。
- **グループ仕様**: デフォルトで `id: "none"` (名前: 「なし」) のグループを許容し、初期状態や未選択時のハンドリングを適切に行う必要があります。
- **循環参照防止**: グループ参照機能があるため、プレビューや実行時に循環参照をチェックし、エラーとして表示する必要があります。

## 正規化（Normalize）機能について
- ステップに **正規化 (NFKC)** が追加されています（`Step.normalize`）。
- 新規グループ作成時、先頭に **正規化ステップが自動挿入**されます。
- 正規化の挙動（記号統一など）は「設定」タブでON/OFFできます（`Settings.normalization`）。

## IPC / Backend 実装の注意点（RenameStep）
- Rust 側の rename 実行は [execute_rename_files](cci:1://file:///c:/codes/regexp-rename/src-tauri/src/lib.rs:178:4-249:5) で行います。
- `steps` は [RenameStep](cci:2://file:///c:/codes/regexp-rename/src/types/index.ts:39:0-41:25) の union/enum（`regex` / `normalize`）として送ります。
- 実行時は `normalization` も一緒に送って、Rust 側でも同じ正規化が適用されます。

## テスト実行の注意点（Vitest）
- Windows環境で worker 起動が不安定な場合があるため、[vitest.config.ts](cci:7://file:///c:/codes/regexp-rename/vitest.config.ts:0:0-0:0) で `pool: 'threads'` を使用しています。

## コードの編集・追加時
- コードの編集・追加時、相関するテストコードも編集・追加する
- コードの編集・追加時、相関するドキュメント(README.mdなど)も編集・追加する

## スタイルガイド
- Tailwind CSS を使用し、一貫性のあるモダンな UI を構築します。
- Vue ファイルは `<script setup lang="ts">`, `<template>`, `<style lang="scss" scoped>` の構造を維持します。

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/fabled--) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
