## weread

> This file provides guidance to AI Agent when working with code in this repository.

# SKILL.md

This file provides guidance to AI Agent when working with code in this repository.

## 项目概述

WeRead 是一个微信公众号文章定时爬取工具（Skill）。它定期爬取用户订阅的公众号文章，按时间窗口和关键词筛选，输出结构化 Markdown。**AI 总结由你来直接完成，无需配置外部 API。**

## 核心架构

```
用户文章链接 → 提取biz/fakeid → 微信MP API拉文章列表 →
Playwright WebKit抓正文 → 关键词粗筛 → 输出Markdown → AI 总结
```

- `seek/auth.py` — 扫码登录、提取token/cookie、从文章URL提取公众号fakeid
- `seek/crawler.py` — 微信API拉列表、Playwright WebKit抓正文和图片
- `seek/scheduler.py` — APScheduler定时调度、时间窗口计算、输出到 Blog/{文章目录}/、自动清理（>100个文件夹时删最旧50个）
- `seek/models.py` — 数据模型 (Article, Subscription, TokenStore)

## Agent 操作与常用命令

作为 AI Agent，请使用你可用的终端工具（如 `run_in_terminal`）执行以下命令，完成相应操作：

```bash
python main.py qrcode   # 生成登录二维码到 data/wx_qrcode.png，不阻塞
python main.py login    # 生成二维码 + 等待扫码 + 初始化fakeid（阻塞，最长5分钟。建议使用 async 模式运行）
python main.py once     # 立即爬取单次内容 → 输出 data/articles/articles_*.md
python main.py start    # 启动后台定时调度（应作为长期任务 async 运行）
python main.py test     # 检查配置和认证状态，若失败需重新 login
```

## 环境准备

```bash
pip install -r requirements.txt
playwright install webkit
```

## WeRead 交互式配置向导

当用户表示想要设置、配置或使用 WeRead 时，按以下流程引导。

### 第一步：了解用户需求

询问：
1. 用户打算订阅哪些微信公众号？（让用户提供任意一篇文章链接）
2. 用户对什么主题感兴趣？（帮助提炼关键词和关注点描述）

### 第二步：配置订阅

直接帮用户编辑 `config.yaml` 的 `subscriptions` 部分：

```yaml
subscriptions:
  - name: "公众号名称"
    url: "https://mp.weixin.qq.com/s/xxxxx"    # 该公众号任意一篇文章链接
    keywords:                                    # 粗筛关键词（可选，为空则全部保留）
      - "关键词1"
      - "关键词2"
    interest: "自然语言描述关注点，AI总结时会以此为准判断相关性并提取重点"
```

- `url`：必须是 `mp.weixin.qq.com/s/` 格式的完整文章链接，用于提取公众号身份
- `keywords`：先做关键词粗筛再交给AI；可为空
- `interest`：AI 在总结时参照此描述判断相关性、提取重点信息

### 第三步：登录认证

**如果你是 AI/Agent 环境（例如 OpenClaw, AstrBot等），请执行以下操作：**

1. 使用终端工具以 **后台/异步模式 (async)** 运行 `python main.py login` 命令。
2. 稍等几秒钟，确保进程生成二维码后，使用**图像读取工具 (如 `view_image`)**读取 `data/wx_qrcode.png`。
3. 获取二维码图像后，及时展示或发送给用户，并提示用户使用微信扫码（5分钟内有效）。
4. 继续监控后台命令的输出，直到用户扫码完成并且系统提示登录成功及 fakeid 初始化完成。
5. **注意**：用户必须是某个微信公众号的**管理员或运营者**才能成功扫码登录。

### 第四步：测试与总结

作为 AI，你需要主动执行并跟进：
1. 运行 `python main.py once` 等待爬取结束。
2. 成功后，使用文件读取工具查看输出的 `data/articles/articles_*.md` 文件。
3. **完成总结任务**：
   - 认真通读提取出的文章正文。
   - 根据用户在 `config.yaml` 中配置的 `interest`（用户关注点），判断文章是否真正具有相关性。
   - 对每篇强烈相关的文章撰写 100-300 字的中文摘要（提炼核心主题、关键观点以及其与用户关注点的直接关联）。
   - 按公众号分组，梳理出结构化、条理清晰的阅读简报。
   - 将最终的简报通过对话发送给用户。

### 第五步：启动定时运行

使用终端异步执行以下命令启动守护进程：
```bash
python main.py start
```

时间窗口规则（默认）：
- 10:00 → 前一日 22:00 ~ 今日 10:00
- 14:00 → 10:00 ~ 14:00
- 18:00 → 14:00 ~ 18:00
- 22:00 → 18:00 ~ 22:00

## 定时执行后的 AI 处理流程 (Agent 监控指南)

当后台的定时任务触发完成爬取后，`Blog/index_*.md` 会被更新。作为 Agent，你应该定期检查或在收到更新提示后执行以下流程：

1. 检测到新的 index Markdown 文件。
2. 使用文件读取工具 (`read_file`) 读取该索引文件，了解本期新增了哪些文章，以及它们对应的存放目录。
3. 对每篇疑似相关的文章，进入 `Blog/{文章目录}/` 并读取内部的 `article.md` 正文内容。
4. **图像理解增强**：如果该目录下存在图片，主动调用**图像查看工具 (`view_image`)** 理解活动报名二维码、活动海报、流程图等信息。
5. 综合文本与图像信息，依据用户配置的关注点，筛选出最相关的 1-2 张核心图片。
6. 构建深度简报（简报不仅包含文字摘要，还需插入核心图片并提供一句话配图解析，说明该图与核心内容的直接关联）。
7. 组装完毕后，将简报主动推送给用户。

## Docker 部署

- 浏览器类型必须为 `webkit`，headless 必须为 `true`
- `playwright install webkit` 需在 Dockerfile 中执行
- `data/token.json` 需持久化挂载

## 故障排查

- **fakeid 获取失败**: 重新运行 `python main.py login`
- **API 200003**: token 过期，重新登录
- **API 200013**: 请求频率过高，增大 `schedule.request_interval`
- **0 篇文章**: 检查 fakeid 是否正确（应为 base64 格式如 `Mzk3NTQ4ODA1Ng==`）

---
> Source: [DOGOGOD/WeRead](https://github.com/DOGOGOD/WeRead) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-19 -->
