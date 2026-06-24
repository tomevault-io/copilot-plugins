## aidio

> 基于 LangGraph 的个人 AI 电台项目，能够读取用户听歌习惯，像 DJ 一样为用户选择和播报音乐。

# MinMaxDio - 个人 AI 电台

基于 LangGraph 的个人 AI 电台项目，能够读取用户听歌习惯，像 DJ 一样为用户选择和播报音乐。

## 技术栈

- **框架**: FastAPI + LangGraph
- **AI**: MiniMax (via LangChain OpenAI 兼容接口)
- **TTS**: MiniMax speech-2.8-turbo
- **音乐**: 网易云音乐 API
- **日历**: 飞书日历 API
- **天气**: OpenWeather

## 项目结构

```
minmaxdio/
├── main.py              # FastAPI 入口
├── config.py            # 配置管理
├── graph/               # LangGraph 工作流
│   ├── state.py         # 状态定义
│   ├── nodes.py         # 节点函数
│   └── MinMaxDioAgent.py  # Agent 定义
├── services/            # 外部服务
│   ├── netease.py       # 网易云音乐
│   ├── minimax_tts.py   # MiniMax TTS
│   ├── feishu.py        # 飞书日历
│   ├── weather.py       # 天气
│   └── upnp.py          # UPnP 推送
├── scheduler/           # 定时任务
├── router/              # API 路由
├── state/               # 状态存储
└── prompts/             # 提示词模板
```

## 启动

1. 安装依赖: `pip install -r requirements.txt`
2. 配置环境变量: 复制 `.env.example` 为 `.env` 并填写 API Key
3. 启动服务: `python -m minmaxdio.main`

## API

- `GET /api/now` - 获取当前状态
- `POST /api/action` - 执行动作
  - `{ "action": "chat", "params": { "message": "你好" } }`
  - `{ "action": "play", "params": { "song_id": "123" } }`
- `WS /api/stream` - WebSocket 实时流

## LangGraph 工作流

```
用户输入 → router_node → [简单指令] → action_node
                        → [音乐相关] → netease_node
                        → [自然语言] → context_node → llm_node → speak_node
```

---
> Source: [hairushi/aidio](https://github.com/hairushi/aidio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-24 -->
