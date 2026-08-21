## dsh-remote-web-gateway

> 本仓库实现 DeepSeek Harness 的远程透明 Web 网关。本文件是仓库内工作的最高优先级操作规范；产品范围、安全边界或外部发布发生冲突时停止并交由人工负责人决定。

# 项目开发规范

本仓库实现 DeepSeek Harness 的远程透明 Web 网关。本文件是仓库内工作的最高优先级操作规范；产品范围、安全边界或外部发布发生冲突时停止并交由人工负责人决定。

## 产品边界

- 网关透明代理现有 DSH Web UI，不重做聊天界面，不设置业务功能白名单，不主动隐藏或删除 DSH 功能。
- DSH 与网关源站都只监听回环地址；公网入口只能是经过 HTTPS、身份认证和加密隧道的边缘入口。
- 认证必须覆盖所有 HTTP 请求和 WebSocket upgrade；健康检查使用独立回环端口，不占用 DSH URL 空间。
- 不修改 DSH 官方源码来绕过边界。优先使用官方 Web profile、网关适配或独立最小兼容补丁。
- 不自研密码算法。JWT、TLS 和隧道使用维护良好的标准实现。

## 开发流程

1. 依次读取 `docs/PROJECT_STATUS.md`、`docs/ARCHITECTURE.md`、`docs/THREAT_MODEL.md`、`docs/COMPATIBILITY.md` 和相关决策。
2. 先核实本地 `D:\dev\deepseek-harness` 固定版本源码，再修改代理假设。
3. 精确修改；不顺手重构、不触碰真实 DSH home、会话、凭据或公网账户。
4. 网络、安全、协议和生命周期行为必须有自动化测试；测试使用假上游、假 JWT、临时密钥和回环端口。
5. 变更后运行相关单测、集成测试、类型检查、构建和敏感信息扫描；同步更新项目状态与兼容性清单。
6. 只有人工负责人可以批准域名、Cloudflare/Tailscale/VPS 配置、真实密钥、外部发布和最终验收。

## 代码与安全

- TypeScript ESM、`strict`，Node.js `^22.19 || >=24`；避免 `any`。
- 所有部署参数在启动时严格校验，错误必须 fail closed。
- 禁止接受非回环监听地址或非回环 DSH 上游；不得提供关闭认证、跳过验签或放宽同源检查的生产开关。
- 不记录 JWT、Cookie、Authorization、查询字符串、请求体、真实邮箱、公网 IP、密钥或 DSH 内容。
- 代理前删除身份断言和转发身份头；只向 DSH 传递其业务所需的浏览器请求。
- 不提交 `.env`、Tunnel token、私钥、真实域名、真实账号标识或用户会话数据。

## 完成定义

交付报告包含目标、改动文件、实际测试结果、源码兼容性证据、未解决问题、风险和文档更新。缺少安全验证、真实移动浏览器证据或项目状态更新时，不得宣称项目完成。

---
> Source: [summer1238/dsh-remote-web-gateway](https://github.com/summer1238/dsh-remote-web-gateway) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-21 -->
