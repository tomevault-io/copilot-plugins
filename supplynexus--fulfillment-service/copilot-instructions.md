## external-api-constraints

> 编辑 Printify/Shopify 调用时参考——limit 上限、service 层 cap


# 外部 API 约束（Third-party API Limits）

调用 **Printify / Shopify** 等第三方 API 时，必须先查官方文档的 **limit、分页、rate limit**，在 **service 层** 做上限保护，并在本 rule 中记录，避免重复踩坑。

## Printify API

| 接口 | 约束 | 说明 |
|------|------|------|
| `GET /v1/shops/{id}/products.json` | **limit ≤ 50** | 每页最大 50，否则 400 |
| `GET /v1/shops/{id}/orders.json` | **limit ≤ 50** | 每页最大 50，错误码 8150 |
| 批量/分页 | 同上 | `get_orders` / `get_orders_batch` 内用 `min(limit, 50)` |

- 任务或 API 层传入的 `limit` 若可能 >50，必须在 **PrintifyService** 内 cap，勿直接传给 Printify。
- 详见 **printify-integration.mdc**。

## Shopify API（Admin GraphQL）

| 用法 | 约束 | 说明 |
|------|------|------|
| `orders(first: N)` | **first ≤ 250** | Connection 单次请求通常最大 250，具体以 [Shopify GraphQL 文档](https://shopify.dev/docs/api/usage/rate-limits#graphql) 为准 |
| `lineItems(first: N)` 等子 connection | 按文档 | 多数 connection 有独立上限（如 50/250），不要传过大 first |
| REST Admin API 分页 | `limit` 按端点 | 各端点不同，需查对应文档 |

- 新增或修改 Shopify 请求时，先查该 connection/endpoint 的 **max page size**，在 service 层用 `min(requested, max)`，默认参数勿超过文档上限。
- 避免出现类似 Printify 的「任务默认 limit=100、接口只支持 50」导致 400。

## 开发约定

1. **新接第三方 API**：在文档里确认 limit/page 上限，在封装层统一 cap，并在本 rule 或对应 integration rule 中写清。
2. **改 limit 或分页**：检查是否超过该接口上限，必要时加 `min(limit, MAX)`。
3. **报 400/422 与 limit 相关**：优先检查是否违反上表，修正后更新本 rule 防止再犯。

---
> Source: [supplynexus/fulfillment-service](https://github.com/supplynexus/fulfillment-service) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-13 -->
