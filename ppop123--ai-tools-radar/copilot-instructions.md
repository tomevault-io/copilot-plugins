## ai-tools-radar

> 把这个项目丢给 AI 时，读这个文件就够了。

# AGENTS.md — 给 AI agent 的操作手册

把这个项目丢给 AI 时，读这个文件就够了。

## 跑起来（唯一需要做的事）

```bash
cd <本目录> && python3 -m http.server 8899
# 浏览器打开 http://127.0.0.1:8899/
```

- 纯静态，无依赖、无构建、无 npm。
- 必须用 HTTP 服务打开；`file://` 直接双击打不开数据（fetch 限制）。
- 验证方式：`curl -s http://127.0.0.1:8899/data/data.json | head -c 200` 返回 JSON 即正常。

## 结构

- `index.html` — 全部 UI（原生 JS，无框架）。四个视图：总榜/增长榜/新品雷达/外链库。
- `data/data.json` — 站点数组。每行字段：
  `domain, name, desc_zh, desc_en, categories[], free, signup, visits, clicks, bl, bl_blog, global_rank, sem_traffic, sem_positions, mix{organic,direct,…}, monthly[[YYYY-MM-DD,visits]…], mom, kw[{n,v,c}], listed_month, n_dirs, registered, organic, dr`
- `data/library.json` — 外链库页面数组：`url, src, title, plat, ascore, nt, targets[{d,a}], seen`
- `data/links/<domain>.json` — 单域 dofollow 明细：`[{u,s,a,p,s2,f}]`（u=来源页,s=标题,a=锚文本,p=平台,s2=权重分,f=首见 epoch 秒）
- `data/links/index.json` — 有明细的域名清单（字符串数组）。

## 改 UI 时的注意点

- index.html 里 fetch 路径全部以 `data/` 开头，移动文件要同步改。
- 数据文件加了 `?t=<timestamp>` 防缓存，别去掉。
- 新视图加进 `VIEWS` 数组 + `SORTS` 映射 + I18N 双语键（zh/en 都要）。
- 截图自测（有 playwright-core 的话）：起服务后访问四个 tab 各截一张。

## outreach/（外链投放工具）

用户看完数据要投放时用这个（LLM-in-the-loop 管道，详见 outreach/README.md）：

```bash
cd outreach && npm install   # playwright-core;另需本机 Chrome 或 npx playwright install chromium
cp my_site.example.json my_site.json     # 可选 capsolver_key;信箱走 agently-cli,不在此配
cp kit.example.json kit.json             # 产品资料包(填表槽位/forbidden_claims 红线)
cp identities.example.json identities.json  # persona 池
export LLM_ENDPOINT=... LLM_KEY=... LLM_MODEL=...
python3 targets.py && python3 driver.py --limit 5
```

**开工前确认用户已准备**（缺了别跑）：OpenAI 兼容 LLM 端点（LLM_* 环境变量）、
收信信箱（两条腿至少通一条，都免费：agent.qq.com 走 agently-cli `auth login`；
agentmail.to 拿 API key 填 my_site.json 的 agentmail_* 字段 + `pip install agentmail curl_cffi`）、
persona 身份池（identities.json）、产品资料包（kit.json）。
mail_sweeper.py 是生产文件逐字复制的最小改动移植，改它先读文件头移植说明。

- kit.json / identities.json / my_site.json 全是占位模板，必须替换成用户真实信息，别用示例值投
- 先 `driver.py --limit 5` 小批验证，没问题再放量；state.jsonl 是唯一状态源，别手改
- 提交后验证邮件由 `mail_sweeper.py --loop` 自动处理（agently-cli 收信+LLM 判意图+点验证链接，
  四条安全闸别动）；先 `--dry-run` 演一遍再放--loop
- 验证码站没配 capsolver_key 会标 manual 进 human_tasks.jsonl，人工处理，不要尝试自动过码
- delivery_ambiguous = 提交可能已投达但终局未定，永不自动重投，只能人工裁决
- pending_review ≠ 上线：终核器 `verify_link.mjs --pending --update-status` 确认在线且
  dofollow 才抬 success（offline_confirmed 连续 ≥3 次才写 failed——单次核验不判死，
  unknown 不动）；建议每周跑一次
- LLM 端点/打码 key/代理全走环境变量或 my_site.json；私仓的任何 key/产品资料不得进本目录

## 数据更新

本仓库只含数据快照。`scripts/` 下的聚合脚本（build_data / build_link_library / build_links_split）
演示了聚合逻辑，但它们读的是私有数据湖（backlinks-v2/datasets），外部跑不了。
要换自己的数据：按上面的 JSON 字段格式生成 `data/data.json` 即可，UI 不用动。

## 免责声明

流量/排名/反链数据为第三方服务估算值，仅研究参考，别当精确值引用。

---
> Source: [ppop123/ai-tools-radar](https://github.com/ppop123/ai-tools-radar) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-16 -->
