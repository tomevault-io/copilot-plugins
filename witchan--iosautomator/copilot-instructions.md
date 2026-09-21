## iosautomator

> - 架构保持独立配置 App + launchd 守护进程，通过用户指定的 iOS MCP 服务进行自动化。

# 项目约束

- 架构保持独立配置 App + launchd 守护进程，通过用户指定的 iOS MCP 服务进行自动化。
- 用户明确要求：不要向 UIKit、目标应用或 SpringBoard 注入代码；不要添加 Hook、方法替换或注入型 tweak。
- 配置 App 自身可以使用 UIKit 绘制界面。系统提示使用系统提供的进程间接口，不靠注入实现。
- 不承诺自动化无法被第三方应用检测。外部 iOS MCP 服务端的内部实现不属于本仓库，不能未经核查就宣称它符合上述约束。

---
> Source: [witchan/iOSAutomator](https://github.com/witchan/iOSAutomator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-21 -->
