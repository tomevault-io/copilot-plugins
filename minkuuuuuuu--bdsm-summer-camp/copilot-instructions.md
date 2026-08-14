## bdsm-summer-camp

> 这份文件是执行说明。目标是让不熟悉服务器的用户把仓库交给你后，也能尽快得到一个可访问、可持续对话的 BDSM 夏令营。

# 给接手仓库的 AI

这份文件是执行说明。目标是让不熟悉服务器的用户把仓库交给你后，也能尽快得到一个可访问、可持续对话的 BDSM 夏令营。

## 开始前

按顺序完整阅读：

1. `README.md`
2. `AI_SETUP.md`
3. `docs/ARCHITECTURE.md`
4. `docs/PROVIDER_PROTOCOL.md`
5. `docs/DEPLOYMENT.md`

先检查仓库当前状态和用户已有条件，再修改。不要要求用户先理解技术术语；能从现有配置、文档和只读检查判断的事情，自己判断。

## 先判断接入场景

- 有 AI 后端和记忆系统：接入两者；
- 只有 AI 后端：接入 AI，设置 `MEMORY_PROVIDER=off`；
- 只有记忆系统：保留记忆，另外配置或实现一个 AI Provider；
- 两者都没有：用服务端的 OpenAI-compatible Provider 接入用户选择的模型接口，先关闭长期记忆。

如果用户的 AI 后端不是 OpenAI Chat Completions 兼容格式，只修改或新增 `server/providers/ai.py` 中的适配器，不要把供应商逻辑散落到业务代码。记忆系统同理，只在 `server/providers/memory.py` 增加适配器。

## 不可破坏的产品约束

- 项目名称是“BDSM 夏令营”；
- 主 AI 不得在代码里固定名字，名称由用户填写，留空时显示“AI”；
- 保留“说话 / 行动 / 想法 / 提问 / OOC”五种参与方式；
- 保留 BDSM Wiki 的 BDSM 101、Theory、Disciplines 三栏完整目录、搜索和来源链接；
- 保留共学与体验两种模式；共学以 AI 教用户为产品外壳，体验保持情境沉浸；
- 保留可关闭、可调语气和活跃度的文字 AI 观众；绝不调用摄像头；
- 主 AI 与观众上下文必须隔离。观众不得读取主 AI 人设或长期记忆，也不得写入记忆；
- 无法按 `X-Camp-Purpose` 隔离的网关，应使用 `AUDIENCE_AI_*` 独立观众端点或关闭观众；独立端点不得继承主 AI 密钥、额外请求头或额外请求体；
- 不在应用层加入“不要太露骨”“保持教育语气”“只推进一步”等削弱成人体验的隐藏提示；
- 除最小模式引导、用户参与方式和可选原有人设/记忆外，主题上下文只使用当前 BDSM Wiki 原词条内容；
- 页面必须能持续对话、保存和恢复会话，并能返回选课目录。

## 密钥与隐私

- 所有密钥只放在服务端 `.env` 或部署平台的秘密变量中；
- 不把 `.env`、数据库、缓存、人设文件、日志或真实聊天记录提交到 Git；
- 不把后端地址做成 `NEXT_PUBLIC_*`，除非用户明确需要浏览器直连且理解暴露后果；
- 默认优先使用同域 `/api` 转发；
- 不在健康检查、报错或日志中返回密钥和原始上游响应；
- 多人共用前，不得把当前本地 `visitor_id` 当成真正身份认证。

## 最小改动原则

优先通过 `.env` 完成接入。只有协议不兼容时才新增 Provider。不要为了接入一个用户的后端而改写前端、会话数据库或提示词结构。不要加入与首版无关的 MCP、账号系统、支付或插件市场。

## 完成前必须验证

```bash
pnpm install
pnpm run build
node --test tests/rendered-html.test.mjs
python -m unittest -v server/test_app.py
python scripts/check_setup.py
```

随后做一次真实冒烟测试：打开首页、选择词条、建立共学会话、发送五种输入中的至少一种、刷新并恢复会话。开启观众时确认观众明确标注为 AI；关闭观众时确认不会生成观众消息。

如启用记忆，再验证：主 AI 能召回并写入；观众请求没有任何记忆调用。

## 向用户交付时只需要说清楚

1. 访问地址；
2. 怎样开始和恢复会话；
3. 模型与记忆是否已接通；
4. 数据和密钥存放在哪里；
5. 如何启动、停止与更新；
6. 仍未完成或需要用户决定的事项。

不要向用户展示真实密钥，也不要声称没有实际测试过的部分已经可用。

---
> Source: [Minkuuuuuuu/BDSM-Summer-Camp](https://github.com/Minkuuuuuuu/BDSM-Summer-Camp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-14 -->
