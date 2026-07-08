## muscle-map-2d

> このファイルはこのリポジトリで Codex / Codex が作業する際の運用ルールです。

# AGENTS.md

このファイルはこのリポジトリで Codex / Codex が作業する際の運用ルールです。

---

## プロジェクト概要

筋肉疲労マップ 2D — 完全再設計 v2.0（2026-04-23 開始）

旧実装（v1.1）は Firebase quota 超過・疲労履歴のセッション限定などの問題があり、スキーマ・状態管理・テスト基盤を含めて作り直す。

## 設計ドキュメント（実装時の単一ソース）

**v3.0（現行）の単一ソースは `docs/v3/` 配下:**
- [docs/v3/function.md](./docs/v3/function.md) — v3.0 確定スコープ
- [docs/v3/design.md](./docs/v3/design.md) — v3.0 詳細設計（v2.0 との差分 D1〜D8）
- [docs/v3/02_work-plan.md](./docs/v3/02_work-plan.md) — ブランチ計画・統合順序

**v2.0（凍結・歴史的記録）:**
- [docs/function.md](./docs/function.md) / [docs/design_basic.md](./docs/design_basic.md) / [docs/design_detail.md](./docs/design_detail.md)
  — v3.0 の design.md で上書きされていない箇所のみ有効。

**迷ったら docs/v3/ を読む。AGENTS.md より設計書が優先。**

## テックスタック

Next.js 14 (App Router) / TypeScript / Tailwind CSS / Firebase (Auth + Firestore) / TanStack Query + Zustand / Vitest + React Testing Library + MSW / Vercel (hnd1) / GitHub Actions

---

## 実装時に守る不変ルール

1. **16筋肉対称モデル** — `MUSCLE_IDS` に沿う（`shoulders_left`/`shoulders_right` 等）。種目カタログは `MuscleGroup` 型（左右未分化）で書く（設計書 §1）
2. **Firebase Admin SDK は singleton** — `src/lib/firebase/admin.ts` の `adminAuth`/`adminDb` を import。Route Handler から `initializeApp` を呼ばない（§19-2）
3. **API Route は全て `withAuth` 経由** — 認証ヘルパーを使わない Route Handler を作らない（§15）
4. **スライダーは `[確定]` ボタン保存のみ** — debounce 自動保存は実装しない（§12-2）
5. **体図の色はクライアント側で再計算** — `useFatigueWithDecay` を使い、Firestore read を増やさない（§12-5）
6. **Firestore 書き込みは batch** — リセット・ワークアウト保存は `writeBatch` 必須（§12-1）
7. **時刻の扱い** — 保存は UTC（Firestore `Timestamp`）、表示は JST 固定（`date-fns-tz`）（§14）
8. **Route Handler は必ず Node.js Runtime + 東京リージョン** — ファイル先頭に `export const runtime = 'nodejs'; export const preferredRegion = 'hnd1';`（§19-1）

---

## ディレクトリ規約

```
src/
├── app/              # Next.js App Router（ページ・Route Handler）
├── components/       # UI（layout/body-diagram/fatigue-panel/workout/ui）
├── hooks/            # TanStack Query カスタムフック
├── stores/           # Zustand（UI 状態のみ）
├── types/            # ドメイン型（domain.ts）
├── lib/
│   ├── firebase/     # admin singleton, client config
│   ├── auth/         # verifyUser, withAuth
│   ├── date/         # JST フォーマット
│   ├── fatigue/      # 回復計算・色マップ・筋肉定義
│   └── workout/      # 疲労値インパクト計算
└── test/             # Vitest setup, MSW handlers

data/                 # シードデータ（exercises.json）
scripts/              # seedExercises.ts
```

## よく使うコマンド

| 用途 | コマンド |
|------|---------|
| 開発サーバー | `npm run dev` |
| lint | `npm run lint` |
| テスト | `npm run test` |
| 型チェック | `npm run typecheck` |
| ビルド | `npm run build` |
| 種目シード投入 | `npm run seed:exercises` |
| Firestore ルール/インデックスデプロイ | `firebase deploy --only firestore:indexes,firestore:rules` |

---

## やらないこと

- debounce によるスライダー自動保存
- テストで Firestore を直接モック（MSW で API 層をモックする）
- Edge Runtime の使用（Admin SDK が動かない）
- Route Handler から `initializeApp` を直接呼ぶ
- localStorage/sessionStorage への疲労値保存（Firestore が単一ソース）
- ワークアウトの編集 API 実装（v3.1 以降）。※削除 API は v3.0 スコープ（docs/v3/function.md V-B6）
- 機能追加を勝手にスコープに入れる（v3.0 は `docs/v3/function.md` に記載されたものが全て）

---

## Git 運用ルール

### ブランチ戦略

```
main            ← 本番（v1.1 が最後の稼働バージョン。完成時にここへマージ）
 └─ ver3.0      ← 再設計の統合先（長期ブランチ、develop 相当）
      ├─ feat/xxx
      ├─ fix/xxx
      ├─ chore/xxx
      ├─ test/xxx
      └─ docs/xxx
```

- 完成時に `ver3.0 → main` を PR 経由でマージ
- feat/* は短命（1機能 = 1ブランチ = 1PR）
- `ver2.0` は旧統合ブランチ（凍結）。新規の feat/* は `ver3.0` から切る

### コミットメッセージ（日本語統一）

- **日本語で簡潔に。1行目 50字以内。英語 prefix は付けない**
- 過去形ではなく現在形（「追加した」ではなく「追加」または「追加する」）
- 必要なら空行の後に本文で「なぜ」を書く

**例:**
```
ドメイン型と筋肉定数を追加

16筋肉の左右対称モデル（設計書 §1）に従う。
MuscleGroup は種目カタログ用の論理グループとして別型で定義。
```

```
applyWorkoutToFatigue の並列 read を実装
```

```
useFatigueWithDecay フックで体図の色をクライアント側再計算
```

### PR フロー

1. `ver3.0` から `feat/xxx` を切る（例: `feat/domain-types`, `feat/muscle-diagram`, `fix/slider-initial-value`）
2. 実装 → push → GitHub で **`feat/xxx → ver3.0`** の PR を作成
3. CI（lint + test + typecheck + build）パス確認
4. **squash merge** で `ver3.0` に取り込む（履歴を線形に保つ）
5. マージ後、feat ブランチは削除
6. v3.0 ブランチ PR の完了条件は `docs/v3/02_work-plan.md` §2 の各ブランチ記載に従う

### PR タイトル・本文テンプレート

タイトル: コミットメッセージと同じルール（日本語、prefix なし）

本文:
```markdown
## 概要
何をしたか（1-3行）

## 関連設計書
- docs/design_detail.md §X

## テスト
- [ ] unit
- [ ] component
- [ ] 手動確認（該当すれば）

## スクリーンショット
（UI 変更のみ）
```

### マージ済みブランチの掃除

```bash
git branch --merged ver3.0 | grep -E '^\s+(feat|fix|chore|test|docs)/' | xargs git branch -d
```

---

## v3.0 実装マイルストーンとブランチ計画

v3.0 の作業単位は Step 0〜6 ではなく、[docs/v3/02_work-plan.md](./docs/v3/02_work-plan.md) §2 のブランチ計画とする。各ブランチは `ver3.0` から分岐し、`ver3.0` へ PR を作成する。

| マイルストーン | 内容 | 対象ブランチ |
|----------------|------|--------------|
| M0 | 品質ゲート回復 | `chore/v3-ci-branches`, `fix/workout-datetime-jst` |
| M1 | 仕様正式化 | `docs/v3-spec` |
| M2 | 正しさ | `feat/api-dto-types`, `feat/error-feedback`, `feat/workout-decay-at-performed`, `feat/workout-impacts-persist`, `feat/exercises-catalog` |
| M3 | 性能・防御 | `feat/fatigue-current-doc`, `feat/firestore-rules-v3` |
| M4 | UX | `feat/slider-current-value`, `feat/mobile-layout`, `feat/history-chart-v2`, `feat/workout-optimistic` |
| M5 | 拡張 | `feat/workout-rpe`, `feat/today-recommend`, `feat/workout-delete`, `test/e2e-smoke` |
| M6 | 統合検証・一括リリース | `ver3.0 → main`、rules/indexes デプロイ、seed、本番スモーク |

| 順 | ブランチ | 対応項目 | 主な依存 |
|----|---------|----------|----------|
| 1 | `chore/v3-ci-branches` | CI push 対象修正 | なし |
| 2 | `fix/workout-datetime-jst` | 実施日時初期値の JST 修正 | なし |
| 3 | `docs/v3-spec` | v3.0 仕様正式化 | なし |
| 4 | `feat/api-dto-types` | API DTO 型分離 | 3 |
| 5 | `feat/error-feedback` | Toast・取得失敗 UI | 3 |
| 6 | `feat/workout-decay-at-performed` | performedAt 減衰・createdAt タイブレーク | 3 |
| 7 | `feat/workout-impacts-persist` | fatigueImpacts 永続化 | 3 |
| 8 | `feat/exercises-catalog` | カタログ拡充・全件取得・首ラベル | 3 |
| 9 | `feat/fatigue-current-doc` | current 単一 doc 化 | 4, 6, 7 |
| 10 | `feat/firestore-rules-v3` | users/** 直接アクセス遮断 | 9 |
| 11 | `feat/slider-current-value` | スライダー初期値修正 | 3 |
| 12 | `feat/mobile-layout` | モバイルレイアウト | 5, 11 |
| 13 | `feat/history-chart-v2` | 時間軸チャート | 4 |
| 14 | `feat/workout-optimistic` | ワークアウト楽観的更新 | 6, 7 |
| 15 | `feat/workout-rpe` | RPE 強度反映 | 6, 7 |
| 16 | `feat/today-recommend` | 今日のおすすめ | 11 |
| 17 | `feat/workout-delete` | ワークアウト削除 | 9 |
| 18 | `test/e2e-smoke` | Playwright E2E | 12 |

### CI/CD 運用

- CI の push 対象は `main` と `ver3.0`（`ver2.0` は凍結のため対象外）。全 PR でも CI を実行する。
- push 前に `npm run lint && npm run test && npm run typecheck && npm run build` を通す。
- Firestore rules/indexes のデプロイは `.github/workflows/deploy.yml` により `main` push 時のみ実行する。v3.0 開発中の rules/indexes 変更は `ver3.0 → main` の一括リリース時に反映する。
- v3.0 開発中に rules/indexes の先行デプロイが必要になった場合は、ユーザーの明示指示を受けて `firebase deploy --only firestore:indexes,firestore:rules` を実行する。

---

## Codex が作業する際のルール

### 設計書は凍結（Frozen Specification）

**`docs/` 直下の v2.0 設計書 3 点（`function.md` / `design_basic.md` / `design_detail.md`）は凍結。変更しない。**
v3.0 の生きた仕様は `docs/v3/`（`function.md` / `design.md` / `02_work-plan.md`）で、実装中の仕様調整はこちらを更新する（living document。ただし docs 変更は実装コミットと分離）。

- v2.0 設計書 3 点を実装時に勝手に編集しない（typo 修正・リンク修正すら実装 PR に混ぜない）
- 設計書の内容と実装を合わせる方向は **設計書に実装を合わせる**。逆方向（実装に合わせて設計書を書き換える）は禁止
- 設計書の変更が必要になった場合:
  1. 作業を一度止めてユーザーに相談する
  2. 変更理由と影響範囲を明示して **専用の `docs/xxx` PR** を先に出す
  3. その PR がマージされた後で、該当する実装 PR を進める
- 軽微な typo 等も含め、設計書を触るコミットは **実装コミットと分離する**（レビューで差分が埋もれないため）

### 実装品質のルール

- v3.0 は `docs/v3/`（`function.md` / `design.md` / `02_work-plan.md`）を **実装の単一ソース**とする。AGENTS.md はあくまで運用ルール
- テストなしの実装コミットを切らない（テスト対象は実装と同じ PR 内で書く）
- `npm run lint && npm run test && npm run typecheck && npm run build` が通らないコミットをプッシュしない
- スコープ外機能（例: ワークアウト**編集** API は v3.1 以降）を勝手に実装しない。※削除 API は v3.0 スコープ（docs/v3/function.md V-B6）
- 機能追加のアイデアが湧いても `docs/v3/function.md` の範囲外なら **実装せずメモに留める**

### v3.0 ブランチ着手前の確認ルール

各 v3.0 ブランチの実装を開始する前に、必ず次の順序で確認する:

1. **作成・変更予定ファイルを列挙する** — ゼロから書き出し、省略しない
2. **各ファイルが当該ブランチのスコープ内であることを確認する** — [docs/v3/02_work-plan.md](./docs/v3/02_work-plan.md) §2 の対象ファイルと照合する
3. **スコープ外のファイルは絶対に作らない** — 「ついでに」は禁止。別ブランチのファイルが必要と判断した場合は作業を止めてユーザーに相談する
4. **ユーザーに列挙内容を提示し、承認を得てから実装を開始する**

旧 Step 0〜6 の対象ファイル表は v2.0 実装時の歴史的記録であり、v3.0 のスコープ判定には使わない。v3.0 では `docs/v3/02_work-plan.md` §2 を唯一の照合先とする。

---
> Source: [MakiMakiSalmon/muscle_map_2D](https://github.com/MakiMakiSalmon/muscle_map_2D) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
