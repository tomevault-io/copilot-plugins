## ft

> - センシティブな内容（社名・個人情報・本業の詳細）は書かない（Public リポ）

# Fine-tuning Experiments

## Prohibited Actions

- センシティブな内容（社名・個人情報・本業の詳細）は書かない（Public リポ）
- 学習データに個人情報を含めない
- 生の API キー・シークレットをソースコードにハードコードしない
- ローカルパス（`/Users/...`）を含むファイルはリモートにプッシュしない

## Role

Apple Silicon ローカルで LLM の Fine-tuning を実験するリポジトリ。
MLX + LoRA で小規模モデルの SFT を行い、AIエンジニアリングの実装力を証明する。

## 技術スタック

- **FTフレームワーク**: MLX (Apple Silicon ネイティブ)
- **手法**: LoRA / QLoRA
- **モデル**: Gemma 3 4B (4bit量子化), Qwen3-0.6B (4bit量子化)
- **データ**: AIエンジニアリング面接Q&A（自作データセット）
- **環境**: MacBook Pro (Apple Silicon)

## ディレクトリ構成

```
chat.py              # Gemma 3 4B 対話テスト
chat_qwen3.py        # Qwen3-0.6B 対話テスト（Reasoning対応）
prepare_data.py      # 学習データ準備
eval.py              # 評価スクリプト
learning_curve.py    # 学習曲線分析
merge_adapters.py    # LoRA アダプタのマージ
data/                # 学習データ（JSONL）
adapters/            # 学習済み LoRA アダプタ
```

## 開発コマンド

```bash
make chat          # Gemma 3 4B で対話
make chat-qwen3    # Qwen3-0.6B で対話（Reasoning対応）
make prepare       # データ準備
make train         # LoRA FT 実行
make eval          # 評価
make merge         # アダプタマージ
```

## Guidelines

- 新しいモデルを追加する場合は `chat_{model_name}.py` の命名規則に従う
- 新しいコマンドを追加する場合は `Makefile` にターゲットを追加する
- 学習データは `data/` 配下に JSONL 形式で配置する
- 学習済みアダプタは `adapters/` 配下に配置する

## 今後の拡張候補

- DPO（Direct Preference Optimization）の実装
- Qwen3-0.6B の LoRA FT
- Gemma 3 4B vs Qwen3-0.6B の比較実験

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/hiro8ma)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/hiro8ma)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
