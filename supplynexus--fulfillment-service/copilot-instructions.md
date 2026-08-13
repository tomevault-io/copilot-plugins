## printify-integration

> 编辑 Printify 相关代码时参考——凭据、limit 50、店铺

# Printify 集成（按需参考）

- 凭据：`external_systems` + `get_decrypted_credentials`；历史脚本或见 `tenants/{tenant}/printify/`。
- API limit：商品/订单列表 limit 最大 50（否则 400/8150）；PrintifyService 内 `min(limit, 50)`。
- 店铺：已连接即 sales_channel ≠ "disconnected"；列表 GET shops.json，Header Bearer token。Token 过期在外部系统/tenant 下更新。

---
> Source: [supplynexus/fulfillment-service](https://github.com/supplynexus/fulfillment-service) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-13 -->
