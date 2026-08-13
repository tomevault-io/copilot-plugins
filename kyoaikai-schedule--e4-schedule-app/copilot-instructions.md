## e4-schedule-app

> generateSchedule() ← L1037付近

# CLAUDE.md - Claude Code 用プロジェクト設定

## プロジェクト概要

4階東病棟 勤務表作成システム（React + TypeScript + Vite + Supabase + Vercel）

## 技術スタック

- **フロント**: React 18 + TypeScript + Tailwind CSS
- **ビルド**: Vite
- **DB**: Supabase（PostgreSQL）
- **デプロイ**: Vercel
- **ライブラリ**: xlsx（Excel読み書き）, lucide-react（アイコン）

## 開発コマンド

```bash
npm run dev      # ローカル起動 http://localhost:5173
npm run build    # ビルド（tsc && vite build）
npx tsc --noEmit # 型チェックのみ
```

## ファイル構成

```
src/
  WardScheduleSystem.tsx   # 4階東病棟勤務表システム（約5,175行）★メイン
  App.tsx                  # WardScheduleSystemを直接レンダリング
  lib/supabase.ts          # Supabase クライアント
```

## WardScheduleSystem.tsx の構造

- departmentName: '4階東病棟'
- dbPrefix: 'e4'（Supabaseテーブル名プレフィックス）

### generateSchedule 関数（自動生成アルゴリズム）

```
generateSchedule() ← L1037付近
├── 準備
│   ├── exReqs 構築（希望データ、0-based index）
│   ├── lockedCells 構築（希望+前月データのセルをSet登録）
│   └── isLocked(nurseId, day) ヘルパー定義
├── buildBase() × 30候補 → 最良選択
│   ├── 前月制約反映
│   ├── 希望反映（2パス: 直接希望→夜勤派生）
│   ├── 休日ランダム配置
│   ├── 夜勤割り当て
│   ├── 日勤割り当て
│   ├── 空きセル埋め
│   └── 日勤補充
├── フェーズ2: SA（焼きなまし法）
├── フェーズ3: 公平性調整
├── フェーズ4: 最終強制修正（A〜G）
└── 検証レポート & 保存
```

### データフロー

```
requests[monthKey][nurseId][day(1-based)]
  → exReqs[nurseId][day(0-based)]  ← 希望データ

prevMonthConstraints[nurseId][day(1-based)]  ← 前月引き継ぎ

lockedCells[nurseId] = Set<day(0-based)>  ← 上書き禁止セル
isLocked(nurseId, day) → boolean
```

## 絶対遵守ルール

### データ保護

- **希望（exReqs）は絶対に上書きしない** → isLocked() でガード
- **前月データ（prevMonthConstraints）は絶対に上書きしない** → isLocked() でガード
- セルに書き込む前は必ず `!isLocked(n.id, d)` をチェック

### 4階東病棟 制約条件

| 制約 | 値 | 備考 |
|---|---|---|
| 最小休日数 | 8日/月 | **「休」+「有」のみ。「明」は除外** |
| 最大連続勤務 | 3日 | |
| 平日日勤者数 | 6〜8人 | |
| 土日祝日勤者数 | 5人 | |
| 夜勤後パターン | 夜→明→休 | 必須 |

### 休日カウント（最重要）

```
✅ 休日: 「休」「有」
❌ 休日でない: 「明」「管明」（勤務扱い）
```

## 注意事項

- `isLocked` は generateSchedule 内のローカル関数と、UI表示部のローカル変数で別々に存在する。混同しないこと。
- generateSchedule 内で `exReqs` や `prevMonthConstraints` を直接条件チェックする箇所は `isLocked()` に統一済み。新たなセル書き換えロジックを追加する場合も必ず `isLocked()` を使うこと。
- `hasReq` / `reqAt` は旧修正（126e060）の残骸。存在してはならない。

---
> Source: [kyoaikai-schedule/e4-schedule-app](https://github.com/kyoaikai-schedule/e4-schedule-app) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
