## huoguang-voice-assistant

> - 本项目只做 AI 对话与火山引擎文本转语音网站。

# AGENTS.md - 火山语音生成工具

## 项目边界
- 本项目只做 AI 对话与火山引擎文本转语音网站。
- 不读取、不依赖、不修改同级目录里的其它工具。
- 密钥可以放在本项目 `.env`，也可以由用户在浏览器设置里填写；不能写进前端源码或提交到仓库。

## 目录约定
- `src/`：网页界面代码。
- `server/`：本地后端和火山引擎接口调用。
- `tests/`：接口和关键逻辑测试。
- `public/`：静态资源。
- 根目录放运行配置，例如 `package.json`、`.env.example`。

## 命名约定
- 前端组件使用 `PascalCase.jsx`。
- 普通工具函数使用 `camelCase.js`。
- 测试文件使用 `*.test.js`。

## 清理约定
- 生成的临时音频不进项目目录，浏览器直接播放返回结果。
- 不在服务端保留用户输入、对话内容和密钥。
- 发现废弃文件先确认用途，再清理。

---
> Source: [xiansa20/huoguang-voice-assistant](https://github.com/xiansa20/huoguang-voice-assistant) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-02 -->
