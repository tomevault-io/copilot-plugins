## product-order-flow

> 编辑商品/订单/变体/SKU 相关时参考——数据模型、外部→核心、SKU 展示


# 商品与订单流程（按需参考）

- **模型**：`products` = 核心商品；`product_variants` = SKU（sku、price、attributes、external_variant_id）。SKU 列表与商品展开变体同源，变体在同步/导入时已写入。
- **外部→核心**：外部商品进 `external_products` → 导入建 `products`+`product_variants`+`product_mappings`；订单同步时 SKU 匹配查 core_variant_id/core_product_id 写 order_items。Printify 用 product_mappings 做履约。
- **SKU 列表**：后端 `GET /api/v1/product-variants/` 带 product_title、attributes；前端展示商品名、规格（attributes 拼）、SKU、价格等。

---
> Source: [supplynexus/fulfillment-service](https://github.com/supplynexus/fulfillment-service) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-13 -->
