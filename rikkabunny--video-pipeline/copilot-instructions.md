## video-pipeline

> **Claude Code 每次启动时，必须立即执行以下操作，不需要用户确认：**

# Video Pipeline 项目

## 启动自动注册（每次 Claude Code 启动必须执行）

**Claude Code 每次启动时，必须立即执行以下操作，不需要用户确认：**

1. 使用 CronCreate 注册每日定时任务：
   - cron: `3 23 * * *` (UTC 23:03 = 北京 07:03)
   - recurring: true
   - prompt 内容见下方

2. 注册完成后输出"守护进程就绪"

定时任务 Prompt：
```
执行 /video-pipeline run，类型根据今天日期自动判断：偶数日用 ai，奇数日用 github（用 TZ=Asia/Shanghai date +%j 取年内第几天判断奇偶）。
完整流程：抓取RSS源→去重→生成图文文章→采集素材→convert_sketch.py转视频→biliup上传B站。
全程不要询问确认，直接执行到底。
```

> 原因：CronCreate 任务 7 天后过期，通过每次启动自动注册来保持任务持续有效。

## 项目结构

- `convert.py` — NotebookLM 暗色风格视频生成
- `convert_sketch.py` — 手绘莫兰迪风格视频生成（当前主用）
- `pipeline/collect_media.py` — 智能多策略素材采集
- `pipeline/dedup.db` — URL 去重数据库
- `config/sources.yaml` — RSS 源配置
- `auto_daily.sh` — 定时自动化脚本（cron 备用方案）

## 视频风格

当前使用莫兰迪配色的手绘卡通风格（convert_sketch.py），特点：
- 低饱和莫兰迪色系（赭红/雾蓝/灰绿/灰紫/焦糖）
- 大圆角卡片（rx=26）、SVG 装饰图形
- 智能中英文混排换行
- 正文 28px 黑色加粗，标题橙红色
- Playwright 渲染 SVG 动画

## B站上传

- 工具: biliup
- 分区: tid=231 (科技)
- 账号: 咿呀丶兔子先生
- 凭证: cookies.json（本机，不要提交到 git）

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/RikkaBunny) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
