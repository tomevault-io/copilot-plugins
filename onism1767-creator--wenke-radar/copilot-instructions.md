## wenke-radar

> > 面向 AI 结对开发与人类贡献者的项目地图。推送样式规范见 `docs/推送格式.md`。

# CLAUDE.md — 文科生求职雷达 开发指南

> 面向 AI 结对开发与人类贡献者的项目地图。推送样式规范见 `docs/推送格式.md`。

## 一、这个项目是什么

面向非技术/泛商业方向求职者的岗位增量雷达：每天抓 45 个招聘源官方接口，
按画像过滤，只报今日新增，微信推送。设计哲学：**增量优先、数据可信、用户不被打扰**。

## 二、代码分层（依赖只向上，禁止反向/环形）

```
config.py   纯配置数据（画像/源开关/守卫阈值/各公司参数），无逻辑
   ↓
domain/     领域层：models.JobItem(数据契约) + classify(类别推断) + enrich(JD解析) + results.FetchResult
            + canonical(跨源去重：canonical_job_id=雇主::平台job_id，校招/社招双feed同岗只推一次)
            + delivery(送达域纯逻辑：PushResult+全渠道失败判定+两道触发守卫真源)
   ↓
scrapers/   抓取层：只负责"从网站拿到 List[JobItem]"，不过滤不碰DB不渲染；
            分页循环设 self.reported_total（完整性对账）；有本地过滤的源另设 self.raw_fetched
   ↓
filters.py  过滤层：画像规则纯函数（方向/城市/届别/排实习/排资深含JD经验）
scoring.py  评分层：可投程度 0-100（过滤答"能不能投"，评分答"多值得投"）
   ↓
store.py    存储层：jobs.db + 新增对比 + 归档 + 三道可信度守卫 + source_health + job_events
   ↓
report.py   渲染层（不直连DB）    tracker.py 投递记录层（applications.db 唯一入口）
push.py     推送层（环境变量 PUSH_KEY/WECOM_KEY，支持多key）；返回结构化 PushResult 供 main 判断送达
ops_guard.py 触发守卫薄入口：workflow 跑抓取前调它(去重+防抢跑)，规则在 domain/delivery，可测
   ↓
main.py     编排层：抓取→FetchResult完整性判定→存储→双渲染→推送
```

自查：一个函数同时"抓数据+判合格"或"算规则+拼Markdown"= 混层，拆开。

## 三、接新源流程

1. 探接口（curl → 浏览器抓包 → hook fetch 抓真实 body；老站常是 form 编码不是 JSON）
2. 能复用平台基类就复用（FeishuAts/Beisen/Baiku/Yili/Moka/LiepinCompany）
3. 社招源必设 `recruit_type="社招"`（否则届别豁免/经验过滤/每日新增全失效）
4. 三处注册：scrapers/__init__.py 的 SCRAPERS、config.ENABLED_COMPANIES、report.COMPANY_ORDER
5. 验收：打印 `set(j.recruit_type)`；抽查岗位性质；跑 `stress_test.py 新源名`；
   跑几天后看 source_health 决定是否配 complete_ratio

## 四、常用命令

```bash
PYTHONUTF8=1 python main.py                  # 本地跑（Windows 必带 PYTHONUTF8=1）
PYTHONUTF8=1 python -m pytest tests/ -q      # 237 个测试，改核心逻辑必跑
PYTHONUTF8=1 python main.py --repush         # 补推：按当前配置把当天已入库岗位重推一次
PYTHONUTF8=1 python tests/record_fixtures.py 某源   # 接口改版后重录 fixture
PYTHONUTF8=1 python stress_test.py 某源      # 新源压测
gh workflow run daily.yml                    # 手动触发云端
```

## 五、必须知道的坑

- 服务端"总数"不一定是可抓总数（北森 Count 虚高）→ 按源配 complete_ratio
- 归档判据是 job_id 集合对比，**不依赖时间戳**（跨端时区不可比）
- jobs.db 不进 git（Actions cache + artifact 快照）；applications.db 进 git，两者别混
- git 多步操作别用 `;` 一把梭；push 确认 `main->main` 再触发云端
- 验收不能只看"抓取完成"，要抽查岗位性质与数量对照官网
- **排除词会静默删数据**：加任何排除词前先全库量化它挡掉多少、抽查该不该挡。
  典型：单列"专家"曾误杀 1079 个对口岗（大厂"XX专家"是 P5~P6 普通职级不是管理层）。
  召回率问题不会自己报错，只会表现成"今天怎么这么少"
- 反爬分寸：只调公开接口、限速、每日一次；硬壁垒（签名/登录）探明即放弃并记录

---
> Source: [onism1767-creator/wenke-radar](https://github.com/onism1767-creator/wenke-radar) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-03 -->
