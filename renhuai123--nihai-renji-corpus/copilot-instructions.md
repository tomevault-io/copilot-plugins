## nihai-renji-corpus

> **不要在提交信息里加 `Co-Authored-By: Claude` 尾注。**

# nihai-renji-corpus（开源仓）

## 提交规范

**不要在提交信息里加 `Co-Authored-By: Claude` 尾注。**

原因：这是公开开源项目，该尾注会让 GitHub 在 Contributors 里显示一个 `claude` 账号，
用户不希望它出现在项目署名中。本仓库从第一个提交起就不带该尾注。

## 数据纪律

- `data/entries.jsonl` 由 `docs/` 六部逐条解析生成，两者条数必须一致（19,294）。
- 判语引文照录 OCR 主源，**不据残字臆改**；错型只登记进 `data/corrections-ledger.jsonl`。
- 门禁 `tools/gate_check.py` 七道（G1–G7）；两条源缺陷豁免见 `tools/g7_exemptions.json`，
  均附抽帧实证，只有监督方可追加。

---
> Source: [Renhuai123/nihai-renji-corpus](https://github.com/Renhuai123/nihai-renji-corpus) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-04 -->
