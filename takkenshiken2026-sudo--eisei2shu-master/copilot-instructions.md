## paid-mock-exam

> 有料模試（note）導線の設置・文言・配色ルール


# 有料模試（note）導線

**正本:** `docs/paid-mock-exam.md`

## 必須

- `site-config.json` の `paidMockExam.url` があるサイトだけ表示。なければ UI 非表示
- **05番**（弱点04の次）+ **結果画面**スコア下 CTA。位置・クラス名はテンプレに合わせる
- バナー色は **`--paid-mock-accent`（#c0392b）**。サイト `theme.accent` は使わない
- 外部リンク: `target="_blank"` + `rel="noopener noreferrer"`

## 設定

`modeTitle` / `modePurpose` / `priceLabel` / `footnote` 必須。`scoreMeta`・`scoreLead` で結果画面文言をサイト別に。

画像バナー（任意）: `bannerImage` + `bannerAlt` を設定するとモード選択・結果画面で画像 CTA を表示。画像は `images/affiliate/{資格略称}-paid-mock-note-banner.webp`（1024×576）。未設定時は従来のテキストカード／ストリップ。

## 作業フロー

1. 変更は **exam-site-shell テンプレ**の `index.html` から（未反映サイトは sync）
2. サイト repo に `paidMockExam` 追加 → `site_config.py` → 目視確認 → デプロイ
3. 回数・価格は note 商品と一致させる（例: メンタル=3回、乙4=2回/35問×2）

---
> Source: [takkenshiken2026-sudo/eisei2shu-master](https://github.com/takkenshiken2026-sudo/eisei2shu-master) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-06 -->
