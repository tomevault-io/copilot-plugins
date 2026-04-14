## presentation-maker

> プレゼンテーション動画を簡単に作成できるアプリを作ります。

# プレゼンテーション動画制作アプリ

日本語で回答してください。

## 概要
プレゼンテーション動画を簡単に作成できるアプリを作ります。

## 参加者
あなたはプロジェクトマネージャー兼開発者、私はプロダクトオーナー兼開発者です。

## ドキュメント
下記の指示に従ってください

- @./GEMINI.md: 全体的な指示

実装が進むたびに適当なタイミングで下記を更新してください

- @./README.md: ユーザー目線での使い方
- @./docs/REQUIREMENTS.md: 要件
- @./docs/SPECIFICATIONS.md: 仕様
- @./docs/DEVELOPMENT.md: 開発環境
- @./docs/COMPONENT_DIAGRAM.md: アプリ内のコンポーネント図。mermaid記法で書く。

## 要望
- 仕様技術
    - windows
    - typescript
    - pnpm
    - fluent-ffmpeg
    - VOICEVOX
- 仕様
    - バッチで実行する
    - input/010__title.md のように1枚のスライドをマークダウン形式で作成する。
    - input/010__title.txt のようにスライドに対応するトークスクリプトのテキストファイルを作成する。
        - __ の前はスライドの番号、後はスライドのタイトルを表す。
        - スライドの番号はトークスクリプトの番号と一致する。タイトルは一致しなくても良い。
    - output/010__title.wav のようにトークスクリプトに対応する音声ファイルを作成する。
    - output/010__title.nosound.mp4 のようにスライドに対応する無音の動画ファイルを作成する。トークスクリプトと同じ長さの再生時間を保つ。
        - 対応するトークスクリプトが無いときはスキップ。
    - output/010__title.mp4 のように音声ファイルと無音の動画ファイルを結合した動画ファイルを作成する。
        - 音声ファイルに対応する無音の動画ファイルが無いときはスキップ。


---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/oharato)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/oharato)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
