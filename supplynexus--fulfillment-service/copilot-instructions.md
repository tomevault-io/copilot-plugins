## celery-async-database

> 编辑 Celery/order_automation 任务时参考——签名与调用方一致；async+DB 用线程内 engine


# Celery / 自动化任务（按需参考）

- **order_automation 任务签名**：与调用方传入的 kwargs 一致。写前看 `automation_scheduler_service.py`、`api/v1/endpoints/automation.py` 的 task_kwargs，至少含 `tenant_id`、`ignore_flags`，否则首次运行易报 unexpected keyword argument。
- **任务内 async + DB**：用独立线程 + 线程内 engine + ThreadAsyncSessionLocal；若用 `asyncio.run()` + 全局 `AsyncSessionLocal` 会报 Future attached to a different loop。可抄 `sync_printify_orders_to_local` 或 `sync_shopify_orders_1min_task` 结构。

---
> Source: [supplynexus/fulfillment-service](https://github.com/supplynexus/fulfillment-service) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-13 -->
