## openspec-schemas

> > 給 Claude Code 在這個 repo 工作時的脈絡指引。維持繁體中文書寫。

# CLAUDE.md

> 給 Claude Code 在這個 repo 工作時的脈絡指引。維持繁體中文書寫。
>
> 關於這個 repo「是什麼、為什麼存在、有哪些 bridges」 → 看 [README.md](./README.md)(英文)或 [README.zh-TW.md](./README.zh-TW.md)(繁中)。
> 本檔聚焦 Claude 在這個 repo 工作時需要知道的**慣例與紅旗**。

---

## 結構約定

```
openspec-schemas/                     ← 本 repo
├── README.md                         ← 英文,GitHub 預設 render
├── README.zh-TW.md                   ← 繁中,有切換連結
├── CLAUDE.md                         ← 你正在讀的(繁中,給 Claude 看)
├── LICENSE                           ← MIT
├── .gitignore
├── .github/workflows/
│   ├── validate-schemas.yml          ← CI 對每個 bridge 跑 openspec schema validate
│   └── version-check.yml             ← 每週驗證 upstream OpenSpec / Superpowers,落後就開 issue
├── docs/
│   ├── roadmap.md / .zh-TW.md        ← 公開 roadmap
│   └── superpowers/
│       ├── specs/                    ← 設計 spec(brainstorming 產出)
│       └── plans/                    ← 實作 plan(writing-plans 產出)
└── superpowers-bridge/                ← 第一個 bridge,自包式 schema bundle
    ├── README.md / .zh-TW.md         ← 完整 bridge 文件(含 install + integration runbook)
    ├── schema.yaml                   ← OpenSpec 讀的 schema 定義
    └── templates/                    ← artifact 模板
        ├── brainstorm.md
        ├── proposal.md
        ├── design.md
        ├── spec.md
        ├── tasks.md
        ├── plan.md
        ├── verify.md
        └── retrospective.md
```

未來新增 bridge:在 repo 根加一個 `<new-bridge>/` 子目錄,內含與 `superpowers-bridge/` 相同結構。CI matrix 在 `.github/workflows/validate-schemas.yml` 的 `matrix.bridge` 加一行即可。

## 命名約定

- **Repo / 目錄 / schema name**:lowercase + hyphen + 對的單複數
  - repo:`openspec-schemas`(複數,可長多個 bridge)
  - bridge dir / schema name:`superpowers-bridge`(單數)
  - 不用 PascalCase(雖然 OpenSpec 自身 repo 用 `OpenSpec`,但他們的 CLI / npm package 都是 lowercase,我們對齊功能性命名)
- **Locale 編碼**:用 `zh-TW`(繁中)、`zh-CN`(簡中);避免裸寫 `zh`

## 雙語策略

| 檔案類型 | 語言 |
|---------|------|
| 入口 `README.md` | 英文 canonical + `README.zh-TW.md` 翻譯 + 頂端切換連結 |
| `CLAUDE.md`(這份) | 繁中(給維護者 + Claude;國際讀者從 README 入口進來) |
| `docs/roadmap.md` | 英文 canonical + `.zh-TW.md` 翻譯 + 切換連結 |
| `superpowers-bridge/README.md` | 英文 canonical + `.zh-TW.md` 翻譯 + 切換連結 |
| `schema.yaml`、`templates/*.md` | 英文(機器讀 + 國際讀者) |
| Commit message | 英文(國際慣例) |
| Code comment | 英文 |

**翻譯同步原則**:英文 canonical,翻譯版可能滯後。修改英文版時若 schema / 工作流發生實質變動,要同步更新繁中版。小改動允許先英文後繁中。

## Schema 修改流程

1. 編輯 `<bridge>/schema.yaml` 或 `<bridge>/templates/*.md`
2. 本地驗證:
   ```bash
   mkdir -p /tmp/test-project/openspec/schemas
   cp -R <bridge>/ /tmp/test-project/openspec/schemas/
   cd /tmp/test-project
   openspec schema validate <bridge-name>
   openspec schemas
   ```
3. 若有時序錯位 / 錯誤行為更動,**也要同步更新 `superpowers-bridge/README.md` 的「六個值得記住的設計觸點」段**(尤其是 verify/retrospective 時序錯位那段),並同步繁中版。
4. Commit message 用英文,符合 conventional commits(`feat:`、`fix:`、`refactor:`、`chore:`、`docs:`、`ci:`)
5. push 觸發 CI

## 三個 alfred-openspec 顧慮的應對(內化記憶)

PR #970 review 提出三個顧慮,本 schema 在 v1 已具體應對。Claude 在這個 repo 修任何 schema 行為前都要記住:

| 顧慮 | 應對 |
|------|------|
| #3 主動 commit 使用者 git | **完全移除**。Step 0 改為 skill PRECHECK,只驗 skill 不動 git |
| #1 與 Superpowers 強耦合無 capability detection | **Layer 1**:每個 invoke skill 的 instruction 開頭跑 PRECHECK,缺失就 STOP。**Layer 2**:對 verify / retrospective 加 evidence-based PRECHECK(`git log`、`grep` 檢查可觀察狀態) |
| #2 verify 時序錯位(以及 retrospective 同型) | 已知限制,在 bridge README 的「設計觸點 #6」文件化。完整修法等 OpenSpec 引擎引入 `post_apply` phase。Layer 2 evidence-based PRECHECK 是當前緩解 |

**修 schema 時的紅旗** —— 以下行為**不要做**(會反 PR #970 的應對):

- ❌ 在 instruction 寫「主動 git add / git commit」
- ❌ 拿掉某個 PRECHECK 但沒換更強的替代品
- ❌ 把 verify / retrospective 從 artifact 拉掉但沒在 README「設計觸點」段同步更新限制
- ❌ 改 schema name 但沒同步改 bridge 內所有文件 + 頂層 README 的 bridge 索引

## 相關連結

- 設計 spec:[`docs/superpowers/specs/2026-05-02-openspec-schemas-monorepo-design.md`](./docs/superpowers/specs/2026-05-02-openspec-schemas-monorepo-design.md)
- 實作 plan:[`docs/superpowers/plans/2026-05-02-phase-1-implementation.md`](./docs/superpowers/plans/2026-05-02-phase-1-implementation.md)
- PR #970 review:<https://github.com/Fission-AI/OpenSpec/pull/970>
- 既有 spec-kit superpowers bridges 參考:
  - [RbBtSn0w/spec-kit-extensions/superpowers-bridge](https://github.com/RbBtSn0w/spec-kit-extensions/tree/main/superpowers-bridge)
  - [WangX0111/superspec](https://github.com/WangX0111/superspec)

---
> Source: [JiangWay/openspec-schemas](https://github.com/JiangWay/openspec-schemas) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-06 -->
