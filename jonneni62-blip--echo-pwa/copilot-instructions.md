## echo-pwa

> 此文件定义了 AI Assistant 在本机工作区执行各类任务与网络调用时的强制代理与直连策略。

# Gemini AI 网络与代理通讯规则

此文件定义了 AI Assistant 在本机工作区执行各类任务与网络调用时的强制代理与直连策略。

请在后续调用终端运行、数据拉取或网络分析时，严格按照以下规则配置环境变量（如 `HTTP_PROXY`, `NO_PROXY` 等）：

## 1. 强制走代理 (Proxy Required)
以下与云端及外网直接交互的功能，**必须**统一通过代理 `http://127.0.0.1:8234` 进行：
- 💬 **与用户的对话通讯**
- 🧠 **代码深度分析与模型推理**
- 👁️ **视觉识别（调用 Gemini Vision API）**

## 2. 严禁走代理 (Direct Connection Only)
以下涉及本地宿主机底层的网络通信，**必须**直连 `127.0.0.1` 环回地址，**严禁使用任何代理**，否则会导致连接阻断：
- 🔌 **MCP 服务器通讯** (Model Context Protocol 通信机制)
- 📁 **本地文件系统的读取与写入**
- 💻 **终端 (Terminal) 侧的本地指令执行**

## 3. 测试环节特殊代理 (Test Cases Proxy)
在执行或拉取特定的开发测试用例时：
- 🎯 **特殊测试端点**：`http://127.0.0.1:4310`
- 🚥 **环境规则**：凡是访问、测试该 4310 端口的网络流量，同样**必须**强制经过代理环境。
    

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jonneni62-blip)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/jonneni62-blip)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
