## tenant-config

> 编辑 tenants/租户/凭据相关时参考——目录结构、安全忽略


# Tenant 配置（按需参考）

- **目录**：`tenants/{tenant_name}/{service_name}/`，下可放 api_token.token、login_info.json、credentials.json（均敏感、不提交）。Git 忽略 *.token、login_info.json、credentials.json、*.key、*.pem；权限 600。
- **读取**：Path("tenants")/tenant_name/service，按需读 token 文件与 login_info.json。脚本可 `--tenant {name}`。凭据不提交、不硬编码；token 到期前轮换。

---
> Source: [supplynexus/fulfillment-service](https://github.com/supplynexus/fulfillment-service) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-13 -->
