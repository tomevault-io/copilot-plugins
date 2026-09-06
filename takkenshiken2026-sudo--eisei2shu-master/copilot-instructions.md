## seo-editorial-rollout

> SEO記事デザイン・内部リンク・執筆品質の全サイト展開テンプレ（exam-site-shell 正本）


# SEO 記事 — 全サイト展開テンプレ

**詳細正本:** `docs/seo-editorial-rollout-template.md`  
**CSS 正本:** `seo-editorial.css` + `SEO_EDITORIAL_CSS_VER` in `tools/seo_editorial_chrome.py`

## 確定デザイン

- H1 **24px** / 要点・関連見出し **19px** / H3・FAQ質問 **19px**
- 要点ボックス = 信頼性パネルと同じ callout（グレー、薄青不可）
- FAQ = `<details class="term-faq-item" open>`（表・1列リスト不可）
- 目次アンカー: `--seo-scroll-anchor-gutter` 24–40px + sticky ヘッダー分
- 関連リンク = **カード型**（左アクセント + → + ホバー薄青）。本文リンク = 青太字下線

## 確定機能

- `tools/internal_links.py` — ハブ導線、用語自動リンク、過去問、関連用語フォールバック
- 執筆: 量産テンプレ禁止（`EDITORIAL_BOILERPLATE_PHRASES` = ERROR）

## 反映時

**手順正本:** `docs/seo-editorial-rollout-checklist.md`

1. Phase 0: テンプレ `verify_seo_editorial_rollout.py` + `build_all`
2. サイトごと: drift → sync → verify → build_all（または checklist §4.2 SEO 最小ビルド）
3. 目視 V1–V5 + 台帳記録
4. commit/push はユーザー明示指示時のみ

---
> Source: [takkenshiken2026-sudo/eisei2shu-master](https://github.com/takkenshiken2026-sudo/eisei2shu-master) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-06 -->
