## no-loopback-web

> 判定・レビュー用 Web は 127.0.0.1 にバインドしない


# 判定 Web はループバックに出さない

この作業機には GUI もブラウザも無い。判定・レビュー用のローカル Web は **既定も例外も `0.0.0.0`（または LAN アドレス）** にする。`127.0.0.1` / `localhost` / `::1` は起動時に拒否する（`scripts/score_server.py` と `scripts/edit_sft_review_server.py` と同じ）。

Makefile の `HOST` 既定も `0.0.0.0`。手順に `http://127.0.0.1:` を書かない。別機体のブラウザから、このマシンの LAN アドレスとポートを開く。

---
> Source: [k16shikano/ja-tech-edit-score](https://github.com/k16shikano/ja-tech-edit-score) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-28 -->
