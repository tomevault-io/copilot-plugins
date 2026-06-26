## raindaytsujigirigoonies

> [novel2hermes_jp](https://github.com/kgmkm/novel2hermes_jp)（Hermes Agent スキル）由来のワークフローを OpenCode で動作するよう再構築したもの。日本語小説の企画・執筆・推敲をエージェントワークフローとして実行する。

# RainDayTsujigiriGoonies — OpenCode Novel Pipeline

## このプロジェクトの目的

[novel2hermes_jp](https://github.com/kgmkm/novel2hermes_jp)（Hermes Agent スキル）由来のワークフローを OpenCode で動作するよう再構築したもの。日本語小説の企画・執筆・推敲をエージェントワークフローとして実行する。

このテンプレートは `~/novel2hermes-template/` から `scaffold.sh` で生成されました。

## アーキテクチャ（novel2hermes_jp からの変更点）

| Hermes Agent 機能 | OpenCode での対応 |
|---|---|
| `fact_store` (vecmemori) | プロジェクト内 `.md` ファイルが正規原典。ファイル読み書きで代替する |
| `session_search` | OpenCode の履歴検索で代替 |
| `memory` (built-in) | OpenCode の知識グラフ（memory tools）で代替 |
| `/compress` | 同等機能なし — エージェント自身がコンテキスト管理する |
| `hermes chat -q -m MODEL --provider` | OpenCode の `opencode` CLI または task カテゴリ切り替えで代替 |

## 前提ツール（環境に合わせて編集）

| ツール | 状態 | 補足 |
|---|---|---|
| **vecmemori** | `~/vecmemori/` にインストール (任意) | FTS5 + ruri-v3 日本語埋め込み |
| **hermes-fake-moa** | `~/.hermes/skills/hermes-fake-moa/` (任意) | MoA並列推敲用 |
| **Python 3.12+** | システム標準 | |
| **NotebookLM CLI** | `nlm` コマンド (任意) | 編集レビュー用 |

## ディレクトリ構造

```
{project-root}/
├── AGENTS.md              ← このファイル
├── opencode.json           ← OpenCode設定
├── Makefile                ← レビュー・パイプライン
├── proposal.md             ← 企画書
├── character/              ← キャラ設定（1ファイル1キャラ）
├── worldbuilding/          ← 世界観設定
├── plot/                   ← プロット（1ファイル1章）
├── novel-tagged/           ← 正規原稿（【!話者:感情!】タグ付き・編集対象）
│   ├── 第1章.md
│   ├── 第2章.md
│   └── en/                 ← 英語版（任意・手動翻訳）
├── novel/                  ← 生成物（strip-tags.py で novel-tagged/ から自動生成）
│   ├── 第1章.md
│   ├── 第2章.md
│   └── en/                 ← 英語版（novel-tagged/en/ からコピー）
├── scripts/                ← パイプラインツール
│   ├── notebook-review.sh  ← NotebookLMレビュー
│   ├── moa-review.sh       ← MoA 4視点レビュー
│   ├── init-tags.py        ← タグ初期投入（heuristic）
│   ├── strip-tags.py       ← タグ除去（novel-tagged/ → novel/）
│   ├── hybrid-tagger.py    ← ヒューリスティック+LLM タグ修正
│   └── generate-illustration.py  ← 挿絵生成（任意）
├── .opencode/agents/       ← OpenCodeカスタムエージェント
│   ├── moa-review.md
│   └── moa-aggregate.md
└── results/                ← レビュー結果
```

### 正規原典のルール

- **`novel-tagged/` が正規原典**（人間・AI の編集対象）。全ての台詞に `【!話者:感情!】` タグが付与されている
- **`novel/` は生成物**。`scripts/strip-tags.py` で `novel-tagged/` から自動生成される。原則として直接編集しない
- 英語版（`novel-tagged/en/`）は手動で翻訳・管理する
- vecmemori は AI の**一次情報取得手段**（高速検索・矛盾検出用）— 利用任意
- 人間が設定ファイルを編集したら、エージェントに「設定ファイルを更新しました」と伝える

## ワークフロー（順序厳守）

### 1. 企画フェーズ（世界観 → キャラ → プロットの順）
- `proposal.md` 作成（タイトル・ジャンル・あらすじ・テーマ・感情曲線・アウトライン）
- 世界観を `worldbuilding/` に保存 → キャラ詳細シートを `character/` に作成 → プロットを `plot/` に保存
- **企画なしに執筆しない。世界なくしてキャラは決まらない。**

参照: `~/novel2hermes_jp/references/planning-workflow.md`

### 2. 執筆フェーズ
- 執筆前に設定ファイル（`.md`）を読んで設定確認
- 三人称過去形。キャラの口調は一貫させる
- 五感ローテーション: シーンごとに視覚以外の感覚を2つ以上
- 比喩: シーンごとに1〜2個。クリシェ回避
- 執筆後、新設定・伏線を `.md` に追記

### 3. タグ付与
- `make script CHAPTER=1` で台詞に `【!話者:感情!】` タグを初期投入（heuristic）
- タグが不十分な場合: `scripts/hybrid-tagger.py` で低確信度ケースを抽出 → LLMで話者・感情を修正
- タグ付き原稿は `novel-tagged/` に保存（これが正規原典）

### 4. 推敲フェーズ
- 論理整合性・文体・時代考証・読者視点の4軸でレビュー
- `make review CHAPTER=1` で MoA 推敲（`novel-tagged/` を読む）
- `make notebook-review CHAPTER=1` で NotebookLM 編集レビュー
- 修正は **`novel-tagged/`** に対して行う

### 5. タグ除去（公開用）
- `python3 scripts/strip-tags.py novel-tagged/ --output novel/` で `novel/` を生成
- `novel/` は直接編集しない

## 参照ファイル

`~/novel2hermes_jp/references/` 以下に全ワークフロー詳細がある:

| ファイル | 内容 | いつ読むか |
|---|---|---|
| `planning-workflow.md` | 企画フェーズ（1-1〜1-6） | 企画開始時 |
| `writing-workflow.md` | 執筆（2-1〜2-3） | 執筆開始時 |
| `revision-workflow.md` | 推敲（Phase A/B/C + MoA） | 推敲時 |
| `moa-manual-orchestration.md` | MoAモデル選択・プロバイダ・実行コマンド | MoA実行時 |
| `character-template.md` | 36項目キャラ詳細シート | キャラ作成時 |
| `metaphor-guide.md` | 比喩ガイド | 執筆時任意 |
| `sensory-rotation.md` | 五感ローテーション | 執筆時任意 |
| `illustration-guide.md` | 挿絵生成（ComfyUI/GPT-Image連携） | 挿絵生成時 |

## スタイル規約

- **文体**: 三人称過去形（キャラの口調は一貫）
- **五感**: 各シーン視覚＋2感覚以上
- **比喩**: 1シーン1〜2個。クリシェ禁止、作品世界内の媒体から選択
- **キャラ名初出**: ふりがな必須（例: `灯（あかり）`）
- **括弧**: `「」`（台詞）、`『』`（強調/作品タイトル）、`（）`（補足/心内）
- **禁止**: 著作権侵害、差別的表現、時代錯誤語彙

---
> Source: [lalate/RainDayTsujigiriGoonies](https://github.com/lalate/RainDayTsujigiriGoonies) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-26 -->
