## secondhandbooks

> from fastapi import APIRouter, Request


# Python/FastAPI 后端规则（二手书助手）

## 路由文件模式
```python
# routes.py 标准结构
from fastapi import APIRouter, Request
from shared.config import get_purchase_conn, dict_from_row, logger

router = APIRouter()

@router.post("/api/purchase/xxx")
async def api_xxx(request: Request):
    body = await request.json()
    try:
        conn = get_purchase_conn()
        # ... SQL操作 ...
        conn.close()  # 必须关闭
        return {"success": True, "data": result}
    except Exception as e:
        logger.error(f"xxx error: {e}")
        return {"success": False, "error": str(e)}
```

## SQL 查询规范
- JOIN 关联表获取名称字段：`LEFT JOIN devices d ON d.id = o.device_id`
- 字段别名匹配前端：`oi.unit_price as price, oi.quantity as qty`
- 结果去重：用 `seen_ids = set()` 去重
- 参数化查询：`conn.execute(sql, params)` 防注入
- LIKE 搜索：`f'%{keyword}%'`

## 常用导入
```python
from shared.config import (
    get_purchase_conn, get_inventory_conn,
    dict_from_row, logger,
    BASE_DIR, MS_DIR, DATA_DIR
)
```

## 后台任务模式
```python
import threading
_jobs: dict = {}
_jobs_lock = threading.Lock()

def _run_job(job_id, items):
    job = _jobs[job_id]
    for item in items:
        if job.get("canceled"): break
        # 处理逻辑
        job["processed"] += 1
    job["done"] = True

# 启动后台线程
t = threading.Thread(target=_run_job, args=(job_id, items), daemon=True)
t.start()
```

## API 响应格式
```python
# 成功
{"success": True, "results": [...]}
{"success": True, "data": {...}}

# 失败
{"success": False, "error": "error_message"}
```

## 常见陷阱
- SQLite `conn` 必须 `close()`，否则锁死
- PaddleOCR 首次加载阻塞事件循环约30秒
- `dict_from_row(row)` 将 `sqlite3.Row` 转为普通 dict
- 文件路径用 `Path` 而非字符串拼接

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/zhouyoukang)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/zhouyoukang)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
