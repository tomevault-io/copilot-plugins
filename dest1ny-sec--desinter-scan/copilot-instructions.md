## desinter-scan

> ├─ Phase 1: 6 个 Collector 串行（资产发现，有依赖）

# interscan v2.0 — Claude 智能编排信息收集系统

## 架构

```
run.py (并行引擎)
  ├─ Phase 1: 6 个 Collector 串行（资产发现，有依赖）
  ├─ Phase 2: 2 个 Collector 串行（服务探测）
  ├─ Phase 3: 14 个 Collector 全并发（深度扫描）
  ├─ Phase 4: 1 个 Collector 可选（screenshot）
  └─ --flat / --agents: 全并发模式，所有 collector 同时跑
         ↓
  SQLite 数据库（实时写入）
         ↓
  Claude 读取 → 智能评估 → 输出优先攻击面
```

## 全并发模式（重要）

当用户已提供资产列表或使用 `--agents` 时，**自动启用全并发模式** — 所有 collector 同时运行，不区分 Phase。因为域名已就位，不存在依赖链。

你不再逐个调用脚本。`run.py` 已经自动完成 Phase 1→2→3→4。
你的工作是在扫描完成后：

1. 读取资产摘要
2. 做多维度价值评估
3. 输出按攻击价值排序的优先目标

## 快速模式 vs 深度模式

**默认快速模式**（用户友好）：
- DNS 爆破 2000 条
- 端口扫描 Top 3000
- 目录扫描中字典（9638）

**深度模式**（`--deep`）：
- DNS 爆破 8000 条
- 全端口扫描（1-65535）
- 目录扫描大字典（45522）

用户感觉扫描太浅时说"加 `--deep` 重跑一次"即可。

## 完整工作流

### 方式一：全自动（推荐）

```bash
# 快速扫描（默认）
python3 run.py --target {{TARGET}}

# 深度扫描
python3 run.py --target {{TARGET}} --deep

# 加上 Phase 4（screenshot）
python3 run.py --target {{TARGET}} --phase4

# 跳过深度扫描（只做资产发现 + 服务探测）
python3 run.py --target {{TARGET}} --skip-phase3

# 断点续扫
python3 run.py --target {{TARGET}} --resume

# 查看扫描进度
python3 run.py --target {{TARGET}} --status

# 检查环境
python3 run.py --check-tools
```

### 方式二：Claude 手动编排（细粒度控制）

```bash
# Phase 1: 资产发现（串行）
python3 -m collectors.enterprise   --target {{TARGET}} --db output/{{TARGET}}.db
python3 -m collectors.subdomain    --target {{TARGET}} --db output/{{TARGET}}.db
python3 -m collectors.dns_enum     --target {{TARGET}} --db output/{{TARGET}}.db
python3 -m collectors.asn_bgp      --target {{TARGET}} --db output/{{TARGET}}.db
python3 -m collectors.cdn_ip       --target {{TARGET}} --db output/{{TARGET}}.db
python3 -m collectors.cloud_assets --target {{TARGET}} --db output/{{TARGET}}.db

# 此时 Claude 判断：子域名够不够？CDN 绕过了吗？云资产有哪些？
# 如果 Phase 1 发现关键漏洞，可以直接跳过 Phase 2-3，深入那个点

# Phase 2: 服务探测（串行）
python3 -m collectors.port_scan     --target {{TARGET}} --db output/{{TARGET}}.db
python3 -m collectors.network_space --target {{TARGET}} --db output/{{TARGET}}.db

# 此时 Claude 判断：Redis/Docker 未授权？优先深入！

# Phase 3: 深度扫描（全并发 — 13 个 collector 同时跑）
python3 -m collectors.http_probe      --target {{TARGET}} --db output/{{TARGET}}.db &
python3 -m collectors.waf_detect      --target {{TARGET}} --db output/{{TARGET}}.db &
python3 -m collectors.dir_scan        --target {{TARGET}} --db output/{{TARGET}}.db &
python3 -m collectors.cms_finger      --target {{TARGET}} --db output/{{TARGET}}.db &
python3 -m collectors.source_leak     --target {{TARGET}} --db output/{{TARGET}}.db &
python3 -m collectors.js_api          --target {{TARGET}} --db output/{{TARGET}}.db &
python3 -m collectors.crawler         --target {{TARGET}} --db output/{{TARGET}}.db &
python3 -m collectors.mobile          --target {{TARGET}} --db output/{{TARGET}}.db &
python3 -m collectors.github_leak     --target {{TARGET}} --db output/{{TARGET}}.db &
python3 -m collectors.nuclei_scan     --target {{TARGET}} --db output/{{TARGET}}.db &
python3 -m collectors.wayback_history --target {{TARGET}} --db output/{{TARGET}}.db &
python3 -m collectors.site_metadata   --target {{TARGET}} --db output/{{TARGET}}.db &
python3 -m collectors.api_probe       --target {{TARGET}} --db output/{{TARGET}}.db &
python3 -m collectors.legacy_hunter   --target {{TARGET}} --db output/{{TARGET}}.db &
wait
```

全部写入同一个 SQLite，不存在写冲突。

## 实时告警

扫描过程中以下发现会**立即告警**（标 ★）：

| 告警 | 触发条件 |
|------|---------|
| ★ Redis 未授权 | 6379 端口 PING 返回 PONG |
| ★ Docker API 未授权 | /containers/json 返回 200 |
| ★ Elasticsearch 未授权 | /_cat/indices 可读 |
| ★ MongoDB 未授权 | 27017/27018 开放 |
| ★ 公开 S3 Bucket | Bucket URL 返回 200 |
| ★ .git 泄露 | /.git/HEAD 返回 200 或 403 |
| ★ GitHub 密钥泄露 | 搜到 AKIA / secret_key / password 等 |
| ★ JS 硬编码密钥 | JS 中匹配到 AK/SK/Bucket/Token 等 |

## Claude 评估流程

### Step 1: 读取统计
```bash
python3 -m core.reporter --target {{TARGET}} --db output/{{TARGET}}.db --mode stats
```

### Step 2: 导出 JSON（如果资产数 < 1000）
```bash
python3 -m core.reporter --target {{TARGET}} --db output/{{TARGET}}.db --mode json --out output/{{TARGET}}_assets.json
```

### Step 3: 如果资产数 > 1000，分层读取
先读 `vulnerable` 和 `cloud` 和 `leak` 三类高价值资产，再读其他。

### Step 4: 生成 HTML 报告（推荐 — 含评分 + 理由）
```bash
python3 -m core.reporter --target {{TARGET}} --db output/{{TARGET}}.db --mode html --out output/{{TARGET}}_report.html --open
```

### Step 5: 多维度评估

| 维度 | 权重 | 评分标准 |
|------|------|---------|
| 服务风险 | 30% | Redis/Docker/MongoDB/ES 未授权→10分, 敏感端口→7分, 普通端口→3分 |
| 泄露严重度 | 25% | GitHub 密钥/.git 可恢复→10分, 备份文件→8分, JS密钥→7分, 配置文件→5分 |
| 云资产暴露 | 15% | 公开S3 Bucket→10分, 受限Bucket→7分, 云函数端点→5分 |
| CMS 漏洞潜力 | 15% | 已知RCE的CMS+旧版本→10分, CMS明确但版本不确定→6分 |
| API/管理面 | 10% | Swagger+Actuator+Druid→10分, 管理后台→7分 |
| 关联度 | 5% | 子域直连IP > CDN后 > 同C段 |

### Step 6: 输出分层报告

```
★★★★★ Critical — 立即处置
  (必含: Redis未授权 / Docker未授权 / 公开S3 / GitHub密钥)
  1. 54.23.x.x:6379 — Redis 未授权 — 可直接写SSH Key [9.8/10]

★★★★ 高危 — 优先深入
  2. admin.example.com — .git源码泄露 + Tomcat 8.5.32 [8.5/10]
  3. api.example.com — Swagger + JS中AK/SK [8.2/10]

★★★ 中危 — 重点排查
  4. mx.example.com — 无WAF邮件服务器 [6.5/10]

★★ 低危 — 常规资产
  7-20. ...

★ 信息 — 持续监控
  21-80. ...
```

## 23 个 Collector 全列表

```
Phase 1 资产发现 (串行):
  ① enterprise     企业产权 WHOIS/ICP
  ② subdomain      14源子域名 + 置换 + DNS爆破（fast 2000 / deep 8000）
  ③ dns_enum       全DNS记录 + AXFR + SPF解析
  ④ asn_bgp        ASN → IP段展开
  ⑤ cdn_ip         CDN检测 + IP反查 + Favicon哈希 ★增强
  ⑥ cloud_assets   crt.sh云搜索 + CNAME分析 + 智能Bucket探测 ★重写

Phase 2 服务探测 (串行):
  ⑦ port_scan      全端口 + UDP + 敏感服务针对性检测 + 自动告警
  ⑧ network_space  FOFA/Shodan/ZoomEye

Phase 3 深度扫描 (全并发 — 13个):
  ⑨ http_probe        批量HTTP + Title + 聚类 + 自动标记后台
  ⑩ waf_detect        WAF/CDN 15+ 特征
  ⑪ dir_scan          7000+9638+45522字典 + 4764平台专属路径
  ⑫ cms_finger        WhatWeb + 30+ 内置指纹
  ⑬ source_leak       .git/.svn/.env/备份/Swagger/Actuator/Druid
  ⑭ js_api            JS密钥 + API端点 + 字典爆破JS路径
  ⑮ crawler           爬虫
  ⑯ mobile            APK下载+解包+硬编码密钥提取 + iOS App探测 + 小程序检测
  ⑰ github_leak       GitHub dork 搜索密钥/配置/内部资产
  ⑱ nuclei_scan       Nuclei CVE扫描 (critical+high)
  ⑲ wayback_history   Wayback Machine 历史URL (隐藏路径/过时API/敏感参数)
  ⑳ site_metadata     robots.txt/sitemap.xml/邮箱/第三方服务/技术头
  ㉑ api_probe         GraphQL内省 + API版本探测 + JS端点验证
  ㉒ legacy_hunter     过期框架检测 (Struts/Axis/WebLogic) + 死端点备份探测 + EOL版本泄露

Phase 4 增强 (可选):
  ㉒ screenshot       批量截图(gowitness)
```

## 关键提醒

1. **Phase 3 是并发的** — 13 个 collector 全并发，不要逐个启动
2. **实时告警优先处理** — 看到 ★ 标记的资产先深入，不等其他扫描
3. **资产 > 1000 时分批评估** — 先看 vulnerable/cloud/leak，再看其他
4. **favicon hash 是个强工具** — 拿到 hash 后去 Shodan 搜 `http.favicon.hash:xxx` 可能找到源 IP
5. **GitHub dork 是被低估的金矿** — `"{target}" filename:.env` 经常直接返回数据库密码
6. **扫描中断也有报告** — 即使 Ctrl+C 中断，也会自动生成 HTML 报告到 `output/<target>_report.html`
7. **快速模式优先** — 默认 fast，觉得不够再加 `--deep`;

---
> Source: [Dest1ny-Sec/Desinter_scan](https://github.com/Dest1ny-Sec/Desinter_scan) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-05 -->
