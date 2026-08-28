## xhs-vault

> 这个项目大部分环节跟 agent 无关：导出清单、抓取、提取字幕、图片 OCR、分类、生成看板，

# 给 agent 的说明

这个项目大部分环节跟 agent 无关：导出清单、抓取、提取字幕、图片 OCR、分类、生成看板，
全是本地脚本，没有模型参与，也不该有。

agent 只做两件事。

---

## 一、帮人装好

用户把仓库交给你，让你替他配起来。按顺序做，每步验证再往下。

```bash
python3 --version     # 需要 3.9+
swiftc -O -o tools/ocr tools/ocr.swift -framework Vision -framework AppKit   # 本地 OCR
which claude          # 缺了也能跑，只是没有增强搜索
```

**有一步你替不了他：导出收藏清单。** 那需要登录态和真人浏览器。
告诉他去看 README 第 0 步，并特别提醒：Chrome 138+ 之后要在
`chrome://extensions` → Tampermonkey 详情里打开「允许用户脚本」这道总闸，
不开的话脚本根本不会注入，面板不会出现。这一步最容易卡住。

清单文件（`data/xhs-favorites*.json`）到位之后，剩下的照 README 跑就行。
抓取脚本默认每条间隔 1.5~3 秒。**不要调小**，那会给小红书的服务器增加负载，也更容易触发风控。

顺手可以帮他做的：

- 按他的收藏内容调 `scraper/taxonomy.py`（**只改这一个文件**，`classify.py` 是引擎）。
  改之前先读 `docs/分类调优.md`，那七条坑每条都实际发生过
- 装开机自启（`install-autostart.command`），省掉每次点启动

---

## 二、接管增强搜索

看板的搜索有两层：

| 层 | 谁在做 |
|---|---|
| 关键词检索 | BM25，纯本地，不需要 agent |
| 增强搜索 | **agent 接管** |

接管的入口是 `serve.py` 的 `/api/ask`，默认调 `claude -p`（模型在 `MODEL` 常量里改）。

有两种接管方式，deep 模式是重点：

**deep（默认）** —— 不预先喂内容，只告诉 agent 库在哪、检索脚本怎么用，
它自己多轮检索、自己决定读哪几条、自己交叉比对。这条路成立的前提是
被调起的 agent **有工具权限**（能跑 Bash、读文件）。`claude -p` 有。

**fast** —— 服务端先用 BM25 筛 8 条、各截 1800 字塞进 prompt。
适合工具权限受限的 agent，或者只想快速拿答案的场景。

### 换成别的 agent

改 `serve.py` 里的 `run_claude()`，它只做一件事：把 prompt 从 stdin 送进去，
把回答从 stdout 收回来。任何支持这个形式的 CLI 都能替换。

deep 模式的 prompt 在 `PROMPT_DEEP`，里面写明了检索手段和回答要求。
其中这两条不要删：

- **只依据笔记内容回答，不要用自己的知识补充事实**
- **收藏夹里没有就直说，不要硬凑**

第二条特别重要。如果搜不到内容时给出一个听起来完整的答案，用户核对一次就会发现，
之后不会再用这个搜索。实测中 Claude 会明确写出「这几条其实回答不了这个问题」，
应该保留这种行为。

---

## 三、别做的事

- 别提交 `data/` 或 `out/`（`.gitignore` 已排除）。里面是用户的私人收藏内容
- 别开放 CORS。deep 模式下 `/api/ask` 能执行 Bash，
  开通配 CORS 等于让用户访问的任何网站都能调它
- 别把抓取间隔调小
- 别在 `taxonomy.py` 里留着用户的个人信息就往公开仓库推

---
> Source: [wkwunju/xhs-vault](https://github.com/wkwunju/xhs-vault) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-28 -->
