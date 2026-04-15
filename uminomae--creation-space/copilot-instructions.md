## creation-space

> - `creation-space` リポジトリ内の SVG ファイル (themes/ domains/) を元に、Gemini を活用して高精度な図解画像を生成する。

# GEMINI.md - creation-space Project Context

## Project Goal
- `creation-space` リポジトリ内の SVG ファイル (themes/ domains/) を元に、Gemini を活用して高精度な図解画像を生成する。
- 外部リポジトリ（kesson-space 等）への依存を排除し、このリポジトリ内で完結する。

## Constraints & Workflows
- **Image Generation Strategy**: 直接の画像生成が API 制限（RESOURCE_EXHAUSTED）により困難な場合、Gemini を使って画像生成AI（Imagen 3, DALL-E 3, Midjourney等）向けの「超高精度プロンプト」を生成する。
- **Prompt Generation**: `gemini-2.0-flash` または `gemini-2.5-pro` を使用し、SVG の内容を極めて詳細な視覚記述に変換する。
- **Output Location**: プロンプトは `creation-space/assets/img/generated/prompts.md` 等に記録し、ユーザーが外部ツールで生成できるようにする。
- **Dependencies**: `creation-space` 内で完結するスクリプト環境を維持する。
- **Source Material**: `assets/svg/` 以下のファイルをインプットとする。

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/uminomae)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/uminomae)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
