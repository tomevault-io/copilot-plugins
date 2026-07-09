## loom

> loom 项目指引与新用户上手引导(Cursor 也原生读 AGENTS.md,此为冗余保险)


# loom — Cursor 规则

loom 是单人零依赖的跨来源工作台账。完整 AI 指引见项目根 `AGENTS.md`。

**新用户首次上手 / 整理历史** → 打开并逐步执行 `ONBOARDING.md`
(配置 → 采集 → 收编散落资料 → 私有云备份 → 主题层完整分类 → 日常)。

铁律:① 不可逆/外发(删除·移动·push·分享)先向用户确认;② 凭证只进 `~/.loom/.env`,绝不入库;
③ 原始数据(csv/xlsx)留本地 `_data/` 不上云;④ 主题分类看条目实际内容、别靠关键词图快,
大规模用逐条判+对抗复核、分完逐主题核对;⑤ 改代码后 `python3 -m pytest tests/test_loom.py`。

---
> Source: [joycastle/loom](https://github.com/joycastle/loom) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-09 -->
