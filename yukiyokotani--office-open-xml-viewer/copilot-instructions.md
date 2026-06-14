## office-open-xml-viewer

> この CLAUDE.md を読んだら、以下を実行してロールを確定せよ:

# CLAUDE.md (monorepo root)

## Worktree 起動時チェックリスト

この CLAUDE.md を読んだら、以下を実行してロールを確定せよ:

1. `pwd` → パスから worktree ロールを判定
   - `.claude/worktrees/pptx` または `ooxml-pptx` → PPTX session（packages/pptx のみ編集可）
2. 各パッケージの `CLAUDE.md` を必ず読む（パッケージ固有の詳細ルール）
3. 他 package のファイルは読み取り OK、編集は禁止

## プロジェクト概要

OOXML (pptx/docx/xlsx) をブラウザ Canvas に描画するライブラリ群。
Rust/WASM parser + TypeScript Canvas renderer 構成。

## ディレクトリ

- `packages/core/` — 共有レンダリングプリミティブ + 共有型
- `packages/pptx/` — PPTX 固有（Session A 所有）
- `packages/docx/` — DOCX 固有（Session B 所有）
- `packages/xlsx/` — XLSX 固有（Session C 所有）

## Git ワークフロー

**複数セッションが並列で作業するため、main への直接 push は禁止。**

- 作業は必ず feature branch で行う（例: `feature/xlsx-xxx`、`feature/pptx-xxx`）
- `git push origin <branch>` して GitHub で PR を作成し、PR 経由で main へマージする
- `git push origin main` は絶対に行わない（直接 push 禁止）
- **squash merge は使わない。** merge commit（`--no-ff`）または rebase merge を使うこと。squash すると feature branch の commit 粒度が失われ、bisect / revert の単位が粗くなる
- `git push` 前に `git config http.postBuffer 524288000` を設定すること

## 自律作業の原則

- AM1時〜AM9時はユーザー確認不要。破壊的操作以外はすべて自律的に進めること。
- 確認なしで進めてよい作業: コード修正・WASM ビルド・テスト実行・commit/push（feature branch のみ）・Python/npm スクリプト実行。
- 参照画像（`packages/*/tests/visual/references/`）はユーザー指示のみ更新。絶対に自動更新しない。
- pptx/xlsx/docx ファイルは git にコミットしない。

## 実装方針: ヒューリスティックより仕様忠実を優先

- VRT を一時的に良くするためだけのヒューリスティック（「M > 2 なら grid snap」「auto > 720 は atLeast と見なす」「body は natural × M で heading は max(natural, pitch × M)」等）を**入れない**。
  短期的に数字が上がっても別サンプルで後退し、理由を書けない挙動が積み重なる。
- まず ECMA-376 / ISO-29500 の該当節を読み、Word が実際にどう解釈しているか（docGrid の snap ルール、line rule の各意味、paragraph mark sz の扱い、spacing 継承の各属性、compat フラグなど）を突き止める。
- 仕様との差の原因が分からないときは、parser 側で情報を捨てていないか（inherit / merge で潰れていないか）を先に疑う。情報が足りなければ parser を拡張するのが正道。
- 工数が増えても spec に忠実な実装を選ぶ。empirical な定数（1.15、0.25、ceiling 付きの条件分岐など）を入れそうになったら、いったん手を止めて「どの §x.x.x の挙動なのか」を書き出す。書き出せないなら実装しない。
- Excel / PowerPoint / Word の UI 挙動（spec に書かれていないランタイム autofit 等）を reverse-engineering して合わせる場合は、事前にユーザー承認を得ること。迷ったら spec 通りを選ぶ。
- 既存コードに上の原則に反するコードが残っている場合は、触る機会があったら正道に寄せる。
- **ユーザーからの「ここがズレている」「これが描画されない」という指摘は、未実装仕様をあぶり出すヒントとして扱う。**
  指摘箇所だけをピンポイントで合わせ込む（sample-N に合致する定数を入れる、特定パスでのみ仕様を守る、など）対応は禁止。
  まず「どの ECMA-376 / ISO-29500 §x.x.x が関係しているか」「parser がその情報を捨てていないか」を特定し、仕様を完全な範囲で実装する。
- 「特定の条件下でだけヒントを信用する」（例: ruby 段落でのみ `lastRenderedPageBreak` を honor する）系のゲートは、根本となる自前計算の不備を隠蔽するヒューリスティックである。
  短期的にそのゲートが必要な場合は必ずコードコメントで「これはヒューリスティック。§x.x.x の自前計算修正が完了し次第撤去する」と明記し、issue 化して追跡すること。

## コード例の規約

README・ドキュメント・Storybook の public-facing なコード例では以下のルールを守る:

- **型アサーションは `as Type` を使う。後置 `!` (non-null assertion) は使わない。**
  ```typescript
  // OK
  const canvas = document.getElementById('my-canvas') as HTMLCanvasElement;
  // NG
  const canvas = document.getElementById('my-canvas')!;
  ```
- **変数名**: canvas を受け取る viewer (`DocxViewer`, `PptxViewer`) には `canvas`、container を受け取る viewer (`XlsxViewer`) には `container` を使う。

## WASM ビルド手順

```bash
# パッケージ別
cd packages/pptx/parser  && wasm-pack build --target web && cp pkg/pptx_parser_bg.wasm pkg/pptx_parser.js ../src/wasm/
cd packages/xlsx/parser  && wasm-pack build --target web && cp pkg/xlsx_parser_bg.wasm  pkg/xlsx_parser.js  ../src/wasm/
cd packages/docx/parser  && wasm-pack build --target web && cp pkg/docx_parser_bg.wasm  pkg/docx_parser.js  ../src/wasm/

# 全パッケージ一括
pnpm build:wasm
```

## Storybook

Storybook はルートに一本化（port 6006）。各パッケージのストーリーは `packages/*/src/*.stories.ts` に置く。

静的ファイルのパスプレフィックス（`.storybook/main.ts` の `staticDirs` で定義）:
- `packages/pptx/public/` → `/pptx/`
- `packages/xlsx/public/` → `/xlsx/`
- `packages/docx/public/` → `/docx/`

サンプルファイルを fetch する際は必ずプレフィックスを付ける（例: `/pptx/sample-1.pptx`, `/xlsx/sample-1.xlsx`）。

ローカル専用のサンプルストーリーは各パッケージの `Samples.sample.stories.ts` に置き、title は `<Viewer>/Samples` でネストさせる（例: `PptxViewer/Samples`）。`.gitignore` 済みなのでコミット対象外。

```bash
pnpm storybook        # dev server (port 6006)
pnpm build-storybook  # storybook-static/ にビルド
pnpm build:wasm       # 全パッケージの WASM をビルド（Storybook ビルド前に必要）
```

## テスト実行 (VRT)

VRT は **ローカル専用**。CI では走らない（`private/sample-*` が再配布禁止で repo に含められないため）。`demo/sample-*` の reference 画像のみ `packages/*/tests/visual/references/demo/` にコミットされており、`private/*` の reference は手元生成かつ gitignore。

```bash
pnpm build:wasm                 # 必須: 各 parser を最新ソースから再ビルド
pnpm vrt                        # 全パッケージ (pptx + docx + xlsx) の VRT 実行
UPDATE_REFS=1 pnpm vrt          # 現状のレンダリング結果で reference を一括更新
pnpm --filter @silurus/ooxml-pptx vrt   # 単一パッケージ
```

`UPDATE_REFS=1` は意図的に reference を上書きするときだけ使う（renderer 改善後 / 新規サンプル追加時 / 大幅リファクタ後）。日常テストでは付けない。

## リリース手順

ユーザーから「リリースして」と指示されたとき、以下を1つの PR にまとめて実行する。squash merge 禁止ルールに従い、`gh pr merge <N> --merge` でマージする。

1. **README のスクリーンショット更新**（メインタスク）
   - `docs/images/{pptx,docx,xlsx}.png` の 3 枚を撮り直す。
   - Storybook を起動して代表的なサンプルを表示し、Playwright / Claude Preview などでスクリーンショット取得。
   - 構図は既存画像と揃える（viewer + サンプル）。ファイル名は固定。
2. **README の対応表更新**（メインタスク）
   - 前リリース以降にマージされた PR を `git log --oneline` で拾い、機能追加があれば `## Feature Support` の該当行を ❌ → ✅ に反転、または新しい行を追加する。
   - bug fix / 精度向上だけなら対応表は動かさず、根拠は CHANGELOG に書く。
   - **紹介サイトの API リファレンス同期**（メインタスク）: 公開 API（各 `*Viewer` のオプション／メソッド、`*Presentation`・`*Document` の headless API）が前リリースから変わっていたら `site/src/lib/api-reference.ts` を実装に合わせて更新する。型は手動抽出なので放置すると陳腐化する。併せて新フォーマット機能があれば `site/src/components/Capabilities.astro` の該当列にも追記する。サイト自体のデプロイは `v*` タグで `deploy-pages.yml` が自動実行する（site を `/`、Storybook を `/storybook/` に統合配信）。
3. **README 整合性検証**（メインタスク・必須）: タグ公開前に README 全体を実装の現状と突き合わせて検証する。npm の README はバージョン公開時にしか更新されないため、誤記が残ると次リリースまで直せない。最低限、以下を毎回確認する:
   - **Bundle size note / パッケージ形式の記述**: 現在 `@silurus/ooxml` は **ESM 専用（`.mjs`）**。「ES + CJS 合算」「CJS を tree-shaking で落とす」等の古い記述が残っていないか。サイズ数値（数式エンジン ≈3 MB 等）が実態と合っているか。
   - **install / import 例**: `require(...)` ではなく `import`（ESM）になっているか。公開単位が `@silurus/ooxml` 本体＋サブパス（`./docx` `./pptx` `./xlsx`）であることと矛盾しないか（サブパッケージは `private:true`）。
   - **Feature Support 表**: 新機能の行が追加・反転されているか（このリリースの変更と一致するか）。
   - **バージョン依存の記述**: 廃止 API・旧バージョン番号への言及が残っていないか。
4. **CHANGELOG 追記**: `CHANGELOG.md` の先頭に `## 0.x.0 — YYYY-MM-DD` セクションを追加し、docx/pptx/xlsx/charts ごとに 1〜3 行の bullet で要点を書く。ECMA-376 節番号や PR 番号を適宜併記。
5. **バージョン bump**: ルート `package.json` と `packages/{core,pptx,xlsx,docx,markdown,node,vscode-extension}/package.json` の計 8 ファイルを同じバージョンへ揃える。markdown / node は private パッケージだが、バージョンは全パッケージで統一する（リリース番号は単一系列で進める）。VS Code 拡張も npm ライブラリと同じ番号で進めるため、機能変更がない月でも minor を上げる。
6. **PR 作成**: ブランチ名は `release/0.x.0`。PR タイトルは `chore(release): 0.x.0`。マージは必ず `--merge` か `--rebase`（squash 禁止）。
7. **タグ作成**: PR マージ後、main を pull して `git tag -a v0.x.0 -m "v0.x.0"` → `git push origin v0.x.0`。
8. **GitHub Release 作成**: `gh release create v0.x.0 --title v0.x.0 --notes "..."` でリリースノート公開。本文は CHANGELOG の該当セクションを要約し、末尾に `**Full Changelog**: https://github.com/yukiyokotani/office-open-xml-viewer/compare/v0.(x-1).0...v0.x.0` を追記する。既存 v0.12.0 のフォーマットを踏襲すること (`gh release view v0.12.0` で確認可能)。

参照画像（`tests/visual/references/`）はこの手順の対象外。README のスクリーンショットは `docs/images/` 配下のみ。

## VS Code 拡張のリリース

`packages/vscode-extension` は npm ライブラリと**同じバージョン番号**で揃える。`v*` タグを push すると `.github/workflows/publish-vscode-extension.yml` が自動で走り、`vsce publish` が VS Code Marketplace に公開する。`VSCE_PAT` は repo secrets に登録済み前提。

手動で `.vsix` を確認したいときは workflow_dispatch で `dry_run=true` を選ぶと、build と package のみ実行して artifact をアップロードする。

---
> Source: [yukiyokotani/office-open-xml-viewer](https://github.com/yukiyokotani/office-open-xml-viewer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-14 -->
