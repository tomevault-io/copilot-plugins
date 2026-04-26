## image-duplication-check

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## プロジェクト概要

画像の重複チェックツール。指定ディレクトリ内の画像をimgsimライブラリでベクトル化し、全ペアの距離を計算して重複画像を検出・削除する。CLIツールとして `image-duplication-check <directory>` で実行可能。

## セットアップ

```shell
# Python 3.11.6 が必要 (.python-version で指定)
uv sync
```

パッケージマネージャはpoetryからuvに移行済み。

## アーキテクチャ

- `image_duplication_check/__init__.py` — コアライブラリ。画像の読み込み(`load_image`)、ベクトル化(`vectorize_image`, `load_and_vectorize_image`)、距離計算(`image_distance`)を提供
- `image_duplication_check/main.py` — CLIエントリポイント。ディレクトリ指定 → 画像バリデーション → 並列ベクトル化 → 距離計算 → 重複表示・削除

## 主要な依存関係

- **imgsim** — 画像のベクトル化と距離計算（`imgsim.Vectorizer`, `imgsim.distance`）
- **OpenCV (cv2)** — 画像の読み込み（imgsimの依存として入る）
- **torchvision** — imgsimの内部で使用されるモデル

## 重複判定

距離閾値 `< 0.1` で重複と判定（デフォルト値、`--threshold` オプションで変更可能）。

## 使い方

```shell
# 基本
image-duplication-check /path/to/images

# 閾値変更・表示スキップ・自動削除
image-duplication-check /path/to/images --threshold 0.05 --no-show --delete
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Romira915) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-11 -->
