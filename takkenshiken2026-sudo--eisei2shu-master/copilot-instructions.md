## responsive-layout

> Responsive layout — site-pages.css, viewport, mobile breakpoints


# Responsive Layout

**正本:** [docs/responsive-layout.md](../docs/responsive-layout.md)

## 非レスポンシブの典型原因

- **旧 `site-pages.css`**（~1.6k 行、レスポンシブ節なし）→ テンプレ最新へ同期
- 旧 `site-page-header` / `site-theme.css` 未リンク
- サイト別 mobile CSS の追加（**禁止**）

## 正本

| 対象 | ファイル |
|------|----------|
| 静的・生成 HTML | `site-pages.css` … `/* 全ページ共通レスポンシブ */` + `@media (max-width: 760px)` |
| SPA | `index.html` インライン … `@media (max-width: 700px)` |
| 色のみ | `site-theme.css`（**@media 禁止**） |

## 必須

- 全公開 HTML: `viewport` + `site-pages.css`（+ `site-theme.css`）
- body: `site-shell-column-page` + `site-page-wrap`
- 表: `.seo-info-table` は ≤760px でカード化
- 固定フッター: `safe-area-inset-bottom` + wrap の下 padding

## 既存サイト修正

```bash
python3 tools/sync_from_template.py --target /path/to/site
python3 tools/apply_site_config.py && python3 tools/build_all.py
python3 tools/validate_site_integration.py
```

## 目視（375px）

`/`, `articles/index.html`, `terms/index.html`, `q/index.html`, 試験ガイド 1 記事 — 横スクロールなし・topnav 使用可・フッター被りなし

## 禁止

- サイト別 `*-mobile.css`
- 生成 HTML への viewport / CSS 削除
- `site-theme.css` にレイアウト用 `@media`

---
> Source: [takkenshiken2026-sudo/eisei2shu-master](https://github.com/takkenshiken2026-sudo/eisei2shu-master) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-06 -->
