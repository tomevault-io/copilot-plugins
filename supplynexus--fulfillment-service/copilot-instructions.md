## api-id-hashid

> 编辑 API/前端涉及 ID 时参考——对外用 hashid，后端 decode_id


# id_hashid（按需参考）

- 对外：API/前端用 `id_hashid`，不暴露数字主键。后端：`decode_id(hashid)` 接收，`encode_id(id)` 返回；安全响应通常只含 `id_hashid`。
- 接口若可能收到 hashid（查询/路径），应支持 hashid 与数字 id：先 `decode_id(...)`，失败再 `int(...)`。工具：`from app.core.hashids_utils import decode_id`。
- 前端：接口只返回 `id_hashid` 时用其作 value/参数；筛选传 hashid 字符串即可。

---
> Source: [supplynexus/fulfillment-service](https://github.com/supplynexus/fulfillment-service) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-13 -->
