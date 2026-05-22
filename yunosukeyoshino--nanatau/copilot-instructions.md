## nanatau

> This file provides guidance to coding agents when working with code in this repository.


# AGENTS.md

This file provides guidance to coding agents when working with code in this repository.

## Priority Checklist

- 画像生成前に `@.claude/skills/nanatau/STYLE_GUIDE.md` を確認する
- 生成物は必ず `output/` に出す
- 採用した画像だけを `images/` 配下へコピーして正本化する
- 配色は `Pale Butter Yellow (#FFFDE7) / Soft Navy (#2C3E6B) / Warm White (#FFF8E1)` を守る

## Project Overview

VTuber "ななたう" のキャラクターデザイン・素材管理・ブログプロジェクト。

- キャラクター: 魔法少女 x 泣き虫。涙が魔力の源。
- 画像生成: Gemini API (`gemini-3-pro-image-preview` for high-quality, `gemini-3.1-flash-image-preview` for exploration)
- ブログ: Lume (Deno) 静的サイト → GitHub Pages

## Directory Structure

```text
nanatau/
├── images/                # 正本・製本済み素材（git管理）
│   ├── concept/           # 採用キャラ案・キービジュアル
│   ├── icon/              # アイコン素材
│   ├── png-tuber/         # PNGTuber素材
│   │   ├── motion/        # MotionPNGTuber用セット正本
│   │   ├── transparent/   # 透過素材
│   │   └── opaque/        # 非透過素材
│   ├── expression/        # 表情差分
│   └── reference/         # リファレンス・参考資料
├── output/                # nanobanan等の生成出力（.gitignore）
│   └── motion-pngtuber/   # MotionPNGTuber の作業ディレクトリ
├── research/              # 調査資料（.gitignore）
└── blog/                  # Lume ブログ（独自CLAUDE.mdあり）
```

## Rules

- `output/` は生成AIの出力先。作業用なのでgit管理しない
- `research/` は調査スクリーンショット等。ローカルのみ
- 採用した画像は `output/` から `images/` の適切なサブディレクトリへコピーして正本化する
- `images/png-tuber/` は transparent / opaque で分ける
- MotionPNGTuber セットは `images/png-tuber/motion/<set-name>/` にまとめる
- MotionPNGTuber の作業中ファイルは `output/motion-pngtuber/<set-name>/` に置く
- `images/concept/` にはキービジュアルと三面図（reference_sheet）を格納

## Art Style Constraints

全画像生成で守る必須ルール:

- 等身: 5.5-6頭身（ちびでもリアルでもない）
- 線画: クリーンな細〜中太ライン、フラットセル塗りのみ
- 配色: Pale Butter Yellow (#FFFDE7) / Soft Navy (#2C3E6B) / Warm White (#FFF8E1) の3色
- 詳細は @.claude/skills/nanatau/STYLE_GUIDE.md を参照

## Blog Commands

```bash
cd blog && deno task serve   # dev server (localhost:3002)
cd blog && deno task build   # production build
```

## Environment

- `GEMINI_API_KEY` が `.env` に必要（画像生成用）
- `.env` は `.gitignore` に含まれている
- Python 3.13+（キャラシート生成スクリプト用）
  - 依存: `pip install google-genai Pillow "rembg[cpu]"`
- MotionPNGTuber は別系統
  - Python `3.10.x`
  - 依存は repo 直下の `pyproject.toml`
  - 実装本体は `.claude/skills/motion-pngtuber-nanatau/scripts/`
- Deno（ブログ用）

## Git

- Conventional commits: `feat:`, `fix:`, `refactor:`, `docs:`, `chore:`
- Main branch: `main`

---
> Source: [YunosukeYoshino/nanatau](https://github.com/YunosukeYoshino/nanatau) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
