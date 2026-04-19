## life-games

> Farcaster Mini App として動作する、Conway's Game of Life アートプロジェクト。

# Farcaster Infinite Life - プロジェクト CLAUDE.md

## プロジェクト概要

Farcaster Mini App として動作する、Conway's Game of Life アートプロジェクト。
64×64、16色のボードで、ユーザーが「SegmentNFT」（独立作品）を作成し、
共有世界線は「EpochArchiveNFT」（256世代ごと）として別途生成される。

### アーキテクチャ（新案: 2025-12-15〜）

#### SegmentNFT（個人作品）
- **空盤面（世代0）から**ユーザーが注入してn世代進めた「独立作品」
- 共有盤面の状態を参照しない
- **即確定mint**（Pending→Finalizeパイプライン不要）
- Mini Appがクライアント側でGIF生成・即時表示

#### EpochArchiveNFT（共有世界線）
- SegmentNFT mint時に発生する**Contributionログ**を時系列順に適用
- **256世代ごと**にMP4アーカイブとしてmint
- 貢献者（Contributors）は該当Epochを無料mint可能

#### Contribution（貢献ログ）
- SegmentNFTのmint時に自動生成
- `tokenId`, `fid`, `nGenerations`, `cellsEncoded`, `blockNumber`
- 共有世界線は Contribution を順番に適用して構築

### コンセプト
- **SegmentNFT**: 空盤面から始まる個人作品（即確定mint）
- **EpochArchiveNFT**: 全員を繋いだ共有世界線のアーカイブ（256世代ごと）
- **Farcaster連携**: FIDをオンチェーン記録

---

## フォルダ構成（モノレポ）

```
200_LifeGames/
├── CLAUDE.md                    # このファイル（プロジェクトルール）
├── .gitignore                   # Git除外設定
├── .env.example                 # 環境変数テンプレート（Git管理）
├── package.json                 # ルートpackage.json（スクリプト用）
├── pnpm-workspace.yaml          # pnpmワークスペース設定
│
├── packages/
│   ├── webapp/                  # Next.js アプリ（Vercelデプロイ対象）
│   │   ├── src/
│   │   │   ├── app/             # App Router
│   │   │   │   ├── layout.tsx
│   │   │   │   ├── page.tsx     # ホーム画面（ボード表示）
│   │   │   │   ├── buy/         # セグメント作成画面
│   │   │   │   ├── segment/[id] # セグメント詳細画面
│   │   │   │   ├── epoch/[id]   # エポック詳細画面
│   │   │   │   └── api/         # API Routes
│   │   │   │       └── token/[id]/route.ts    # NFTメタデータ
│   │   │   ├── components/      # 共通コンポーネント
│   │   │   ├── hooks/           # カスタムフック
│   │   │   ├── lib/             # ユーティリティ
│   │   │   │   ├── life-engine.ts    # Life シミュレーション（決定論的）
│   │   │   │   ├── color-rules.ts    # 16色カラールール
│   │   │   │   ├── state-encoding.ts # StateV1 エンコーディング
│   │   │   │   └── gif-renderer.ts   # クライアント側GIF生成
│   │   │   ├── providers/       # Context Providers
│   │   │   └── types/           # TypeScript型定義
│   │   ├── public/              # 静的ファイル
│   │   └── package.json
│   │
│   └── contracts/               # Foundry プロジェクト
│       ├── foundry.toml
│       ├── src/
│       │   ├── SegmentNFT.sol   # 個人作品NFT（即確定mint）
│       │   └── EpochArchiveNFT.sol  # 共有世界線アーカイブNFT
│       ├── test/
│       ├── script/
│       └── lib/                 # forge dependencies (Git除外)
│
├── .github/
│   └── workflows/
│       └── epoch-generator.yml  # Epoch生成用 Actions（256世代ごと）
│
├── _backup_life_league/         # 旧プロジェクトバックアップ（Git除外）
│
└── docs/
    ├── requirements/
    │   ├── art/
    │   │   ├── farcaster_infinite_life_mvp_spec.md  # MVP仕様書（旧案）
    │   │   ├── detailed_design.md                   # 詳細設計書（旧案）
    │   │   └── spec_diff_new_plan_segment_independent_epoch_archive_ja.md  # 新仕様差分【必読】
    │   └── battle/              # 旧プロジェクト要件（参照用）
    └── TODO.md                  # 開発TODO
```

---

## Vercelデプロイ設定

| 項目 | 値 |
|------|-----|
| Root Directory | `packages/webapp` |
| Build Command | `pnpm build` |
| Output Directory | `.next` |

---

## 要件定義ドキュメント参照【必読】

実装前に必ず以下のドキュメントを確認すること：

| ドキュメント | パス | 内容 |
|-------------|------|------|
| **新仕様差分** | `docs/requirements/art/spec_diff_new_plan_segment_independent_epoch_archive_ja.md` | **最優先で参照**。新アーキテクチャの詳細 |
| ~~MVP仕様書~~ | `docs/requirements/art/farcaster_infinite_life_mvp_spec.md` | 旧案（参考用） |
| ~~詳細設計書~~ | `docs/requirements/art/detailed_design.md` | 旧案（参考用） |

---

## セキュリティルール【絶対厳守】

### 秘密情報のハードコード厳禁

以下の情報は**絶対に**ソースコードにハードコードしてはならない：

- `EPOCH_MINTER_PRIVATE_KEY` - Epoch mint用秘密鍵
- `GITHUB_TOKEN` - GitHub Actions用
- `ALCHEMY_API_KEY` - Alchemy APIキー
- `BASESCAN_API_KEY` - Basescan APIキー
- その他すべてのAPIキー、秘密鍵、トークン

### 環境変数の管理

```
✅ 正しい方法:
   - .env.local に記載（Git除外）
   - Vercelの環境変数設定で管理
   - GitHub Secretsで管理（Actions用）
   - process.env.XXX で参照

❌ 禁止行為:
   - ソースコードへの直接記載
   - .env.example に実際の値を記載
   - コミットメッセージに秘密情報を含める
   - console.log で秘密情報を出力
```

### コミット前チェック

1. `git diff --staged` で秘密情報が含まれていないか
2. `.env` ファイルがステージングされていないか
3. ログ出力に秘密情報が含まれていないか

---

## 重要な仕様【新案】

### ボードパラメータ

| パラメータ | 値 |
|-----------|-----|
| サイズ | 64×64（4,096セル） |
| パレット | 16色（インデックス 0〜15） |
| トポロジー | Moore近傍（8方向） |
| ルール | **HighLife (B36/S23)** - Birth: 3 or 6, Survival: 2-3 |

### SegmentNFT（個人作品）

| パラメータ | 値 |
|-----------|-----|
| nGenerations | 10〜30 |
| 注入セル数上限 | `floor(nGenerations / 2)` |
| セルエンコーディング | 3バイト/セル（x, y, colorIndex） |
| **起点** | **空盤面（世代0）** ← 共有盤面ではない |
| **mint方式** | **即確定**（Pending/Finalize不要） |

### SegmentNFT保持データ（最小）

| フィールド | 説明 |
|-----------|------|
| `fid` | 購入者Farcaster ID |
| `nGenerations` | 世代数 |
| `cellsHash` | keccak256(cellsEncoded) |
| `rulesetHash` | ルールセットハッシュ（将来互換） |
| `mintedAt` | mint時のblock番号 |

### SegmentNFTイベント

```solidity
event SegmentMinted(
  uint256 indexed tokenId,
  address indexed minter,
  uint256 indexed fid,
  uint8 nGenerations,
  bytes32 cellsHash
);

event SegmentCells(
  uint256 indexed tokenId,
  bytes cellsEncoded
);
```

### EpochArchiveNFT（共有世界線）

| パラメータ | 値 |
|-----------|-----|
| 世代範囲 | 256世代ごと（epoch 0: gen 1-256, epoch 1: gen 257-512, ...） |
| 成果物 | MP4動画 |
| 貢献者 | Contributionを通じて寄与したSegment所有者 |

### Epoch保持データ

| フィールド | 説明 |
|-----------|------|
| `absStartGen` | 絶対開始世代 |
| `absEndGen` | 絶対終了世代 |
| `startStateCID/Root` | 開始状態 |
| `endStateCID/Root` | 終了状態 |
| `artifactURI` | MP4 URI |
| `contributorsCID` | 貢献者一覧JSON |
| `contributorsRoot` | 貢献者Merkle root（任意） |

### ライフルール【絶対厳守】

**HighLife (B36/S23)** を採用。標準Conway's Game of Life (B3/S23) とは異なる。

| 条件 | ルール |
|------|--------|
| **誕生 (Birth)** | 死セルの周囲に **3個または6個** の生セルがあれば誕生 |
| **生存 (Survival)** | 生セルの周囲に **2個または3個** の生セルがあれば生存 |
| **死亡** | 上記以外は死亡（残光なし） |

### カラールール（決定論的）【絶対厳守】

**誕生時**: 親セルから**決定論的に1つ選択**（色インデックスの合計でソート後選択。RGB平均化による中間色への収束を防止）
**生存時**: **自身50% + 隣接平均50%** → 最近接パレット色（隣接色の影響を強めて色変化を促進）
**死亡時**: 即座に死（残光なし）

### 状態エンコーディング（StateV1）

```
aliveBitset: 4,096 bits = 512 bytes（行優先 i = y*64 + x）
colorNibbles: 4,096 * 4 bits = 2,048 bytes（2セル/byte）
合計: 2,560 bytes

stateRoot = keccak256(StateV1Bytes)
```

---

## UIレイアウト規約

### 言語
- **デフォルト**: 英語
- **対応言語**: 英語 / 日本語（切替可能）
- **切替UI**: ヘッダー右上に言語スイッチャー

### 全体レイアウト
- 縦長スマホ画面に最適化（424×695px web / デバイス依存 mobile）
- 100vhを使用してフル画面表示
- 64×64ボードはピンチズーム対応
- **ヘッダー**: ロゴ + 言語切替のみ（ウォレットボタンなし）
- **ナビゲーション**: 下部固定タブナビ（Home, Create, Gallery, My）

### 主要画面（5画面）
| パス | 画面名 | 説明 |
|------|--------|------|
| `/` | Home | 共有世界線の最新状態、最新Epoch、最新Segment一覧 |
| `/buy` | Create | 空盤面にセル配置→GIFプレビュー→即確定mint |
| `/segment/[id]` | Segment Detail | 個人作品表示（クライアント側GIF生成）、シェア |
| `/gallery` | Gallery | Segments一覧 / Epochs一覧 |
| `/my` | My Page | 自分のSegment、無料mint可能Epoch |

---

## 開発ルール

### 技術スタック（固定）

| カテゴリ | 技術 |
|---------|------|
| フロントエンド | Next.js 15 (App Router) |
| スタイリング | Tailwind CSS 4 |
| 状態管理 | React Query + Zustand |
| Web3 | wagmi + viem |
| バックエンド | Next.js API Routes |
| コントラクト | Foundry (Solidity 0.8.24) |
| Epoch生成 | GitHub Actions |
| ストレージ | GitHub Releases / IPFS |
| チェーン | Base Sepolia (開発) → Base Mainnet (本番) |

### コマンド

```bash
# ルートから実行
pnpm dev           # webapp 開発サーバー起動
pnpm build         # webapp ビルド
pnpm forge:build   # コントラクトビルド
pnpm forge:test    # コントラクトテスト

# コントラクトデプロイ（ルートから実行）
# 注意: .env.local はルートにあるため、set -a で環境変数をexportする
set -a && source .env.local && set +a && \
cd packages/contracts && \
forge script script/Deploy.s.sol:Deploy \
  --rpc-url $BASE_SEPOLIA_RPC_URL \
  --broadcast \
  --verify
```

### コーディング規約

1. **TypeScript 必須** - any型の使用は極力避ける
2. **エラーハンドリング** - try-catch で適切にエラー処理
3. **コメント** - 複雑なロジックには日本語コメント
4. **決定論性** - Life エンジン、カラールール、エンコーディングは完全決定論的
5. **命名規則**
   - コンポーネント: PascalCase
   - 関数・変数: camelCase
   - 定数: UPPER_SNAKE_CASE
   - ファイル: kebab-case または camelCase

### Git運用

- `main` ブランチへの直接pushは禁止
- 機能ごとにブランチを作成
- コミットメッセージは日本語可

---

## チェーン情報

### Base Sepolia（開発）

| 項目 | 値 |
|------|-----|
| Chain ID | 84532 |
| RPC URL | https://sepolia.base.org |
| Block Explorer | https://sepolia.basescan.org |

### Base Mainnet（本番）

| 項目 | 値 |
|------|-----|
| Chain ID | 8453 |
| RPC URL | https://mainnet.base.org |
| Block Explorer | https://basescan.org |

---

## Farcaster Mini App 開発ルール【必読】

### LLM/AI向け禁止事項

- ❌ `fc:frame` メタタグを新規実装に使用（レガシー専用）
- ❌ Frames v1 構文の使用
- ❌ マニフェストに存在しないフィールドの作成
- ❌ `"version": "next"` の使用（`"1"` を使用）
- ❌ 2024年以前の古い例の参照

### LLM/AI向け必須事項

- ✅ `fc:miniapp` メタタグを使用
- ✅ `@farcaster/miniapp-sdk` の公式スキーマに準拠
- ✅ `sdk.actions.ready()` を必ず呼び出す
- ✅ マニフェストのドメインは完全一致

### 開発ツール

| ツール | URL |
|--------|-----|
| マニフェスト生成 | https://farcaster.xyz/~/developers/mini-apps/manifest |
| プレビューツール | https://farcaster.xyz/~/developers/mini-apps/preview |
| Embedツール | https://farcaster.xyz/~/developers/mini-apps/embed |

---

## 変更履歴

| 日付 | 変更内容 |
|------|----------|
| 2025-12-13 | 初版作成（Life League） |
| 2025-12-14 | Farcaster Infinite Life に方針転換、CLAUDE.md全面改訂 |
| 2025-12-14 | 詳細設計書追加、画面設計追加、セグメント範囲5-30、多言語対応(EN/JP) |
| 2025-12-14 | ボードサイズ 200×200 → 100×100 に変更 |
| 2025-12-14 | ボードサイズ 100×100 → 64×64 に変更 |
| 2025-12-15 | **新アーキテクチャに移行**: SegmentNFT独立作品化、EpochArchiveNFT導入、Pending/Finalize廃止 |
| 2025-12-15 | コントラクト改修完了、Base Sepoliaデプロイ、デプロイコマンド追記 |
| 2025-12-16 | **HighLife (B36/S23) ルール明記**、カラールール改善（誕生時: 決定論的選択、生存時: 50%+50%）|

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/SaxophoneTrom) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
