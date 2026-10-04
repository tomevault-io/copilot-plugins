## password-manager-switch

> - 工具链版本统一声明在 mise.toml 的 [tools]；优先使用 mise 已安装版本。

# 项目约定

- 使用简体中文。
- 工具链版本统一声明在 mise.toml 的 [tools]；优先使用 mise 已安装版本。
- 项目命令统一定义为 mise task，通过 mise run 执行。
- 不写 UI 单元测试或只复述实现的测试；设置写入、失败恢复等核心行为需要故障场景验证。
- 特权服务仅操作明确列出的三个 secure settings，并允许查询当前用户密码服务的公开元数据；不接收任意命令，不读取密码或通行密钥内容。
- 写入须读取校验；读取失败不得当成空配置；不得把设置写入成功描述为通行密钥业务已经成功。

---
> Source: [cr-zhichen/password-manager-switch](https://github.com/cr-zhichen/password-manager-switch) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-04 -->
