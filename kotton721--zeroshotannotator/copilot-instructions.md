## zeroshotannotator

> 自動クラス生成型アノテーション補助ツール。画像からクラスラベルを自動生成し、セマンティックセグメンテーションマスクを出力する。

# Auto Annotation - プロジェクト概要

自動クラス生成型アノテーション補助ツール。画像からクラスラベルを自動生成し、セマンティックセグメンテーションマスクを出力する。

## アーキテクチャ

2フェーズバッチパイプライン。GPUメモリ節約のため Qwen と SAM3 を同時に載せない設計。

1. **Phase 1 (Qwen)**: vLLM サーバ経由で全画像のクラスラベルを JSON 配列として取得
2. vLLM コンテナを `docker stop` で停止 → GPU メモリ解放
3. **Phase 2 (SAM3)**: 各ラベルごとに1回 SAM3 を実行 → バイナリマスク PNG を保存
4. SAM3 アンロード → GPU メモリ解放

## サービス構成 (docker-compose)

| サービス | イメージ/ビルド | ポート | 役割 |
|----------|----------------|--------|------|
| `vllm` | `vllm/vllm-openai:latest` | 8080 | Qwen3.5-9B を OpenAI互換 API として配信 |
| `backend` | `./backend` (CUDA + Python) | 8000 | FastAPI + SAM3 推論 + パイプライン制御 |
| `frontend` | `./frontend` (Node → nginx) | 3000 | React UI |

## 重要な設計制約

- Qwen3.5-9B は直接ロード不可。必ず vLLM API サーバ経由で使用する
- SAM3 はセマンティックセグメンテーション出力。bbox は出力しない
- SAM3 の出力はバイナリマスク画像（PNG, 白=対象/黒=背景）としてファイル保存
- JSON にはマスク画像のパスを記録する（マスクデータ自体は JSON に含めない）
- 1画像あたり最大10クラス (`MAX_CLASSES = 10`)
- backend コンテナから Docker socket 経由で vLLM コンテナを停止する

## 出力形式

```
output/
├── results.json
├── {image_stem}/
│   ├── {label}.png    # バイナリマスク画像
│   └── ...
```

## 環境変数

| 変数 | デフォルト | 説明 |
|------|-----------|------|
| `VLLM_BASE_URL` | `http://vllm:8080` | vLLM API エンドポイント |
| `QWEN_MODEL_ID` | `Qwen/Qwen3.5-9B` | vLLM で配信するモデル ID |
| `OUTPUT_DIR` | `/app/output` | マスク画像の出力先 |
| `HF_HOME` | `/root/.cache/huggingface` | HuggingFace キャッシュ |

## ビルド・起動

```bash
docker compose up --build        # 全サービス再ビルド＆起動
docker compose build backend     # backend のみ再ビルド
```

## コーディング規約

- コミットメッセージは日本語で書く
- Dockerfile のベースイメージは Debian 系を使う（alpine は使わない）

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Kotton721) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
