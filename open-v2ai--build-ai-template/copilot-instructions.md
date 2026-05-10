## backend-rules

> 后端开发规范和 FastAPI/Python 最佳实践


# 后端开发规范 - FastAPI/Python

## 技术栈

- **框架**: FastAPI
- **语言**: Python 3.12+
- **ORM**: SQLModel
- **数据库**: PostgreSQL + Redis
- **包管理**: uv
- **异步**: asyncio + httpx

## 项目结构

```bash
api/app/
├── core/         # 核心配置 (config, security, i18n)
├── models/       # SQLModel 数据模型
├── schemas/      # Pydantic 验证模式
├── routers/v1/   # API v1 路由
├── crud/         # 数据库 CRUD 操作
├── services/     # 业务逻辑服务
├── dependencies/ # FastAPI 依赖注入
├── utils/        # 工具模块
├── agents/       # AI Agent
└── main.py       # FastAPI 应用入口
```

## Python 编码规范

### 导入规范

所有导入都应在文件顶部，并遵循以下顺序，各组之间用空行分隔：

1. **标准库** (`os`, `datetime`, `typing`)
2. **第三方库** (`fastapi`, `sqlmodel`, `pydantic`)
3. **本地应用模块** (`from app.core...`, `from app.models...`)

**最佳实践**:

- **避免通配符导入**: 不要使用 `from module import *`。
- **使用绝对导入**: `from app.models.user import User`。
- **使用别名处理命名冲突**: `from app.schemas.agent import Agent as AgentSchema`。

## FastAPI 应用配置

应用入口位于 [api/app/main.py](mdc:api/app/main.py)。

```python
from fastapi import FastAPI
from contextlib import asynccontextmanager

# ... 导入路由

@asynccontextmanager
async def lifespan(app: FastAPI):
    """应用生命周期管理，用于初始化和清理资源。"""
    # 启动时操作
    yield
    # 关闭时操作

app = FastAPI(
    title="Build AI Template API",
    version="1.0.0",
    lifespan=lifespan
)

# ... 中间件配置 (CORS)

# 注册 v1 路由
app.include_router(auth.router, prefix="/api/v1")
app.include_router(user.router, prefix="/api/v1")
# ... 其他路由

@app.get("/health")
async def health_check():
    return {"status": "healthy"}
```

## 路由开发规范

路由位于 `api/app/routers/v1/`。

```python
from fastapi import APIRouter, Depends, HTTPException
from typing import List
from ...dependencies.db import get_db
from ...dependencies.auth import get_current_user
from ...models.user import User
from ...schemas.user import UserResponse, UserCreate
from ...crud.user import user_crud
from ...core.exceptions import ForbiddenException

router = APIRouter(
    prefix="/users",
    tags=["users"],
)

@router.get("/", response_model=List[UserResponse])
async def get_users(
    db: Session = Depends(get_db),
    current_user: User = Depends(get_current_user),
    # ... 其他参数
):
    """
    获取用户列表

    - 权限: 管理员
    - 参数: 分页、搜索
    - 返回: 用户列表
    """
    if current_user.user_type != "admin":
        raise ForbiddenException("只有管理员可以查看用户列表")

    users = await user_crud.get_multi(db, ...)
    return users

@router.post("/", response_model=UserResponse, status_code=201)
async def create_user(
    user_in: UserCreate,
    db: Session = Depends(get_db),
):
    """创建新用户"""
    existing = await user_crud.get_by_email(db, email=user_in.email)
    if existing:
        raise HTTPException(status_code=400, detail="邮箱已被注册")

    user = await user_crud.create(db, obj_in=user_in)
    return user
```

## Schema 验证规范

Schema 定义位于 `api/app/schemas/`。

```python
from pydantic import BaseModel, EmailStr, Field, validator
from datetime import datetime

class UserBase(BaseModel):
    """用户基础模型"""
    email: EmailStr
    username: str

class UserCreate(UserBase):
    """用户创建模型"""
    password: str = Field(..., min_length=6)

    @validator('password')
    def validate_password(cls, v):
        # 密码强度验证逻辑
        return v

class UserResponse(UserBase):
    """用户响应模型"""
    id: int
    is_active: bool
    created_at: datetime

    class Config:
        from_attributes = True
```

## 依赖注入规范

依赖项位于 `api/app/dependencies/`。

```python
from fastapi import Depends, HTTPException, status
from fastapi.security import HTTPBearer
from jose import JWTError, jwt
from ..core.config import settings
from ..models.user import User
from ..crud.user import user_crud

security = HTTPBearer()

async def get_current_user(
    credentials: HTTPAuthorizationCredentials = Depends(security),
    db: Session = Depends(get_db)
) -> User:
    """通过 JWT Token 验证并获取当前用户"""
    try:
        payload = jwt.decode(
            credentials.credentials,
            settings.SECRET_KEY,
            algorithms=[settings.ALGORITHM]
        )
        user_id = payload.get("sub")
        if user_id is None:
            raise HTTPException(...)
    except JWTError:
        raise HTTPException(...)

    user = await user_crud.get(db, id=user_id)
    if not user or not user.is_active:
        raise HTTPException(...)

    return user
```

## 服务层规范

业务逻辑服务位于 `api/app/services/`，用于封装复杂的业务流程。

```python
class MembershipService:
    """处理会员相关的业务逻辑"""

    @staticmethod
    async def check_chat_limit(db: Session, user: User) -> bool:
        """检查用户对话限制，超出则抛出 BusinessException"""
        # 1. 获取用户会员信息
        # 2. 检查每日使用量是否需要重置
        # 3. 对比当前使用量和限制
        # 4. 如果超限，抛出 BusinessException
        return True
```

## 异常处理规范

自定义异常位于 `api/app/core/exceptions.py`。

```python
# api/app/core/exceptions.py
from fastapi import HTTPException, status

class NotFoundException(HTTPException):
    """资源未找到异常"""
    def __init__(self, detail: str = "Resource not found"):
        super().__init__(status_code=status.HTTP_404_NOT_FOUND, detail=detail)

class ForbiddenException(HTTPException):
    """权限不足异常"""
    def __init__(self, detail: str = "Forbidden"):
        super().__init__(status_code=status.HTTP_403_FORBIDDEN, detail=detail)

class BusinessException(HTTPException):
    """业务逻辑异常"""
    def __init__(self, detail: str):
        super().__init__(status_code=status.HTTP_400_BAD_REQUEST, detail=detail)
```

在 `main.py` 中配置全局异常处理器来统一错误响应格式。

## 异步和性能

- **异步优先**: 所有 I/O 密集型操作（数据库、网络请求）都应使用 `async/await`。
- **并发执行**: 使用 `asyncio.gather` 并发执行独立的异步任务。
- **缓存策略**: 对不经常变化且查询频繁的数据使用 Redis 缓存。

## 安全最佳实践

- **认证授权**: 使用 JWT，并对需要权限的端点强制执行角色检查。
- **输入验证**: 始终使用 Pydantic Schema 验证所有输入数据。
- **密码安全**: 密码必须哈希处理后存储，切勿明文存储。
- **依赖安全**: 定期使用 `uv` 更新依赖，检查安全漏洞。
- **日志记录**: 记录所有关键操作和安全事件。

---
> Source: [open-v2ai/build-ai-template](https://github.com/open-v2ai/build-ai-template) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-06 -->
