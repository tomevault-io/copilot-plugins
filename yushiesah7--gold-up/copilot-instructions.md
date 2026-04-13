## gold-up

> 本プロジェクトは **DDD + Hexagonal（Ports & Adapters）** を採用します。


# Contributing Guide

本プロジェクトは **DDD + Hexagonal（Ports & Adapters）** を採用します。  
ドメイン（戦略カーネル）は「完全疎結合・純関数」。外部ライブラリは**必ず Adapter/Interface を挟む**こと。

---

## 0) ファイル/ディレクトリ規約
- **1ファイル最大 400 行**（例外：`constants.py` 等の定数・列挙のみ超過可）。
- 層構成（例）：
  - `apps/` … ユースケース（司令塔：`research`, `trade`）
  - `domain/` … エンティティ／値オブジェクト／**Ports（抽象I/F）**
  - `adapters/` … **外部ライブラリの実装（ACL）**：vectorbt / MT5 / DuckDB / pandas-ta など
  - `kernel/` … **戦略カーネル（純関数）**：I/Oなし、乱数なし、未来参照なし
  - `features/` … 指標計算（決定的）、方式固定（例：RSI=Wilder）
  - `spec/` … Pydantic モデルと YAML I/O
- **依存方向**：`apps → domain(Ports) → adapters → externals` の一方向。

参考：Ports & Adapters / Hexagonal（抽象に依存し、具象は外周に隔離）:contentReference[oaicite:2]{index=2}

---

## 1) コーディング規約（DDD 重要ルール）

##
- **外部ライブラリを直接呼ばない**。**必ず Port/Adapter 越し**に利用する（**ACL**で方言・型・例外を吸収）。:contentReference[oaicite:3]{index=3}
- 抽象は **`typing.Protocol`（PEP 544）** で定義（構造的型）。軽量辞書は **`TypedDict`（PEP 589）**。:contentReference[oaicite:4]{index=4}
- DTO/設定は **Pydantic モデル**で境界バリデーション（内部はドメイン用 DTO に正規化）。
- **純関数カーネルは YAML を知らない**：Adapter が `params/features/plan`（プリミティブ値）に変換して渡す。
- **前バー判定→次足Open**の契約は検証エンジン側で：`Portfolio.from_signals(price=open, ...)`。:contentReference[oaicite:5]{index=5}

---

## 2) Lint / Format / Test
- **Ruff** を**リンタ兼フォーマッタ**として使用（`ruff check` / `ruff format`）。設定は `pyproject.toml`。:contentReference[oaicite:6]{index=6}
- **pytest** を使用。設定は `pyproject.toml` の `[tool.pytest.ini_options]` に集約。:contentReference[oaicite:7]{index=7}
- 速度優先：テストは**ユニット最優先**、統合はスモーク＋回帰のみ。

### 便利コマンド（例）
```bash
uv run ruff format
uv run ruff check
uv run pytest -q

## 3) 作業完了後常に実行するコマンド
```bash
uv run ruff check . --fix; uv run ruff format
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/yushiesah7) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
