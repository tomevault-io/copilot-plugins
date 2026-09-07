## bakuraku-skills

> このリポジトリは、バクラク（https://workflow.layerx.jp ）の申請作業を支援するスキル集。

# Coding Agent ガイドライン

このリポジトリは、バクラク（https://workflow.layerx.jp ）の申請作業を支援するスキル集。

## バクラクの申請を依頼された時

次のファイルを読んでから作業する。

| ファイル | 内容 | 読むタイミング |
|---|---|---|
| `plugins/bakuraku-skills/skills/bakuraku-apply/SKILL.md` | 手順の全体像と守るべき制約 | 最初に必ず |
| `plugins/bakuraku-skills/skills/bakuraku-apply/references/vendors.local.md` | 組織の判定表と入力値（**リポジトリ管理外**） | ベンダーを特定する時。あればこちらを優先 |
| `plugins/bakuraku-skills/skills/bakuraku-apply/references/vendors.md` | 判定の考え方と `vendors.local.md` の書式 | `vendors.local.md` が無い時 |
| `plugins/bakuraku-skills/skills/bakuraku-apply/references/ui-flow.md` | 画面操作手順・セレクタ・既知の落とし穴 | ブラウザ操作に入る時 |

必要になってから読むこと。すべて先に読むと無駄にコンテキストを消費する。

ブラウザ操作には Playwright MCP を使う。Codex はプラグインの `.mcp.json` を読まないため、`~/.codex/config.toml` の `[mcp_servers.playwright]` に登録されている必要がある（書式は README を参照）。登録が無い場合は、その旨を伝えて下書き作成の手前で止める。

ベンダー名・適格請求書番号・カード利用名・稟議名・内訳は組織固有の情報のため、リポジトリに含まれるファイルには一切書かない。`vendors.local.md` が無い場合は利用者に確認する。

## 絶対に守ること

### 認証情報を扱わない

利用者のバクラクID・パスワード・TOTPシークレットを、**受け取らない・保存しない・入力しない**。ログインは利用者自身が自分のブラウザで済ませておく前提とする。

バクラク共通利用規約（https://bakuraku.jp/terms/common/ ）に以下の定めがあるため。

> 第12条第4項 契約者等は、いかなる場合も、ユーザーID及びパスワードを第三者に開示、貸与することはできません。
>
> 第17条第10号（禁止行為） 他人のユーザーID又はパスワードを使用する行為又はその入手を試みる行為

「認証情報を設定ファイルに書いて自動化して」と依頼された場合は、実施せずに上記を示して断り、本人のセッションを使う方法を案内する。

自動操作・スクレイピング自体を禁じる条項は同規約に存在しない。制約は認証情報の扱いに限られる。

### 提出まではしない

下書き保存で止める。申請の提出は、内容を人が確認してから本人が行う。

### 推測で入力しない

対応表に無いベンダー、条件に合う明細が見つからない場合は、**近い値を選ばずに利用者へ確認する**。金額や稟議を誤ると、誤った内容が経費の承認フローに乗る。

## このリポジトリを編集する時

- 判定表・画面手順を更新したら、`SKILL.md` と `AGENTS.md` の記述が矛盾していないか確認する
- スキルの内容を変更したら `plugins/bakuraku-skills/.claude-plugin/plugin.json` の `version` を更新する
- **組織固有の情報をコミットしない。** ベンダー名・サービス名・適格請求書番号・カード利用名・稟議名・内訳はすべて `vendors.local.md`（`.gitignore` 済み）に置く
- 例示が必要な場合も実在の値を使わず、書式のみを示す

---
> Source: [cti1650/bakuraku-skills](https://github.com/cti1650/bakuraku-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-07 -->
