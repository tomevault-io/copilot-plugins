## rokid-lingzhu-openclaw

> 灵珠平台接入 - 将 OpenClaw Agent 接入 Rokid 灵珠智能体平台，支持完整 agent session（人格/记忆/工具链）


## 功能

将 OpenClaw Agent 通过 SSE 协议接入 Rokid 灵珠智能体平台，实现：

- Rokid 智能眼镜语音交互 → OpenClaw Agent 完整处理
- 支持完整 agent session（SOUL.md / MEMORY.md / 工具链 / 上下文记忆）
- 流式 SSE 响应，实时返回到眼镜端
- 自动生成鉴权 AK，安全接入

## 支持的设备命令

| 灵珠命令 | OpenClaw 工具名 | 说明 |
|---------|---------------|------|
| `take_photo` | take_photo, camera, photo | 拍照 |
| `take_navigation` | navigate, navigation, maps | 导航 |
| `control_calendar` | calendar, schedule, reminder | 日程 |
| `notify_agent_off` | exit, quit | 退出智能体 |

## 安装

```bash
openclaw plugins install ./extension
```

## 配置

在 `openclaw.json` 中：

```json5
{
  gateway: {
    http: {
      endpoints: {
        chatCompletions: { enabled: true }
      }
    }
  },
  plugins: {
    entries: {
      lingzhu: {
        enabled: true,
        config: {
          agentId: "luna"  // 指定路由到哪个 agent
        }
      }
    }
  }
}
```

## 查看状态

```bash
openclaw lingzhu info    # 查看 SSE 地址和 AK
openclaw lingzhu status  # 查看运行状态
```

## 灵珠平台配置

1. 登录 https://agent-develop.rokid.com/
2. 创建智能体 → 三方智能体 → 自定义智能体
3. SSE 接口地址: `http://<公网IP>:18789/metis/agent/api/sse`
4. 鉴权 AK: 运行 `openclaw lingzhu info` 获取

## 修复记录（相对原版 r-wmi）

- 修复 `registerHttpHandler` → `registerHttpRoute` API 兼容性（OpenClaw 2026.3.x）
- 修复 session 无状态问题：使用固定 session key，支持多轮对话上下文
- 修复 agent 路由：通过 `x-openclaw-agent-id` + `x-openclaw-session-key` header 精确路由
- 支持通过 `config.agentId` 指定目标 agent（默认 luna）

## 致谢

基于 [EndlessJour9527/r-wmi](https://clawhub.ai/EndlessJour9527/r-wmi) 修改，原始协议转换逻辑由 @wmi9527 开发。

---
> Source: [Hylouis233/rokid-lingzhu-openclaw](https://github.com/Hylouis233/rokid-lingzhu-openclaw) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
