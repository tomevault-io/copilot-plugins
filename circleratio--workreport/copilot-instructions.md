## workreport

> このファイルは、このリポジトリで作業する Claude Code (claude.ai/code) 向けのガイドである。

# CLAUDE.md

このファイルは、このリポジトリで作業する Claude Code (claude.ai/code) 向けのガイドである。

## リポジトリ構成

実際のアプリケーションは `app/` 配下にあり、リポジトリルートには設計ドキュメントのみが置かれている。

- `narrative.md` → `REQUIREMENT.md` → `techstack.md` → `SPEC.md` → `PLAN.md` は、
  非公式な要望 → ユーザ要求仕様 → 技術スタック決定 → 画面/データ/IPCの詳細仕様 → 段階的な実装計画
  という順に、徐々に正式化されていくドキュメント群である。UI挙動（ドラッグ/クリックの意味、
  重複予定のレイアウトアルゴリズム、ダイアログ入力項目の規則、配色同期の規則など）の一次情報は
  `SPEC.md` であり、フロントエンドの挙動を変更する前には必ず確認すること
  （例: SPEC.md §4.4の時刻編集時のガード処理や重複予定の列割り当てアルゴリズムは、
  コードだけを読むと見落としやすい微妙な条件がある）。

---
> Source: [circleratio/workreport](https://github.com/circleratio/workreport) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
