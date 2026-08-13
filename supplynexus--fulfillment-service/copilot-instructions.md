## dimension-mapping-design

> 编辑维度/变体映射相关代码时参考——settings.dimension_name_map


# 维度映射设计（按需参考）

- **目的**：Core / Shopify / Printify 维度名不一致（如 Size vs type），匹配按语义、可配置，不写死。存 `external_systems.settings["dimension_name_map"]`：`core_dimension_name → external_dimension_name`，不新增表。
- **使用**：校验 Core↔外部维度一致、变体按维度自动匹配时，用该 map 解析两侧名称；前端优先用接口下发的 map，无则回退单复数/大小写等。Shopify 同法用该连接的 `settings.dimension_name_map`。
- **扩展**：可预留 `dimension_name_map_by_product_type`（按类目）；当前可仅默认 map，落库与保存方式待实现时定。

---
> Source: [supplynexus/fulfillment-service](https://github.com/supplynexus/fulfillment-service) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-13 -->
