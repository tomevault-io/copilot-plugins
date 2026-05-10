## api-route-language-rules

> API 路由语言设置规范


# API 路由语言设置规范

## 核心要求

**所有新建的 API 路由必须支持国际化（i18n），包含中文（zh）和英文（en）语言设置。**

## 实施规范

### 1. 路由函数签名

每个路由端点函数必须导入并使用 `LangDep` 依赖项。

```python
# 必须导入
from app.dependencies.db import LangDep, SessionDep

# ✅ 正确示例
@router.get("/")
async def get_orders(
    db: SessionDep,
    current_user: User,
    lang: LangDep,  # 必须包含语言依赖
    skip: int = 0,
    limit: int = 100,
):
    """获取订单列表"""
    pass
```

### 2. 错误消息国际化

所有用户可见的错误消息都应该使用 `get_message` 进行国际化。

```python
from app.core.i18n import get_message

@router.post("/")
async def create_order(
    order_data: OrderCreate,
    lang: LangDep,
    # ...
):
    """创建订单"""
    try:
        # 业务逻辑
        pass
    except ValueError as e:
        # ✅ 使用国际化错误消息
        error_msg = get_message("order_creation_failed", lang)
        raise HTTPException(status_code=400, detail=error_msg)
```

## 检查清单

- [ ] 每个端点函数都有 `lang: LangDep` 参数。
- [ ] 导入了 `LangDep` 和 `SessionDep`。
- [ ] 错误消息使用 `get_message()` 进行国际化。

## 现有路由示例

参考以下文件的实现方式：

- [api/app/routers/v1/auth.py](mdc:api/app/routers/v1/auth.py)
- [api/app/routers/v1/chat.py](mdc:api/app/routers/v1/chat.py)
- [api/app/routers/v1/admin.py](mdc:api/app/routers/v1/admin.py)
- [api/app/routers/v1/membership.py](mdc:api/app/routers/v1/membership.py)
- [api/app/routers/v1/user.py](mdc:api/app/routers/v1/user.py)

---

**重要提醒**：这是系统级规范，所有开发者在创建新路由时都必须遵守。

---
> Source: [open-v2ai/build-ai-template](https://github.com/open-v2ai/build-ai-template) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-06 -->
