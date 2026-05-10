## database-models

> 数据库模型设计和 SQLModel/Alembic 最佳实践


# 数据库模型设计规范 - SQLModel/PostgreSQL

## 核心原则

- **详细注释**: 所有模型必须有详细的中文注释，说明字段、业务规则和关联关系。
- **应用层维护关系**: 不在数据库层面设置 `FOREIGN KEY` 或 `UNIQUE` 约束，由应用层逻辑保证数据一致性和唯一性。
- **软删除**: 所有表必须包含 `is_deleted: bool` 字段。
- **时间戳**: 所有表必须包含 `created_at` 和 `updated_at` 字段。
- **主键**: 使用自增整数 `id` 作为主键。

## 技术栈

- **ORM**: SQLModel (基于 SQLAlchemy + Pydantic)
- **数据库**: PostgreSQL 15+
- **迁移工具**: Alembic

## 模型文件组织

```bash
api/app/models/
├── __init__.py      # 模型导出
├── user.py          # 用户相关模型
├── chat.py          # 对话相关模型
└── ...              # 其他模型
```

## SQLModel 模型定义规范

### 基础模型模板

所有模型都应包含标准字段，可以通过继承一个 `BaseModel` 来实现。

```python
from sqlmodel import SQLModel, Field, Relationship
from datetime import datetime

class User(SQLModel, table=True):
    """
    用户模型

    业务规则:
    - 邮箱作为唯一登录标识
    - 支持软删除

    关联关系:
    - 一对多: User -> Chat
    - 一对一: User -> UserMembership
    """
    id: Optional[int] = Field(default=None, primary_key=True)
    created_at: datetime = Field(default_factory=datetime.utcnow)
    updated_at: datetime = Field(default_factory=datetime.utcnow, sa_column_kwargs={"onupdate": datetime.utcnow})
    is_deleted: bool = Field(default=False)

    # 基础信息
    email: str = Field(max_length=255, unique=True, index=True, description="用户邮箱")
    username: str = Field(max_length=50, description="用户名")
    hashed_password: str = Field(description="加密后的密码")

    # 账户状态
    is_active: bool = Field(default=True)

    # 关联关系
    chats: List["Chat"] = Relationship(
        back_populates="user",
        sa_relationship_kwargs={"lazy": "selectin"} # 预加载策略
    )
    membership: Optional["UserMembership"] = Relationship(
        back_populates="user",
        sa_relationship_kwargs={"uselist": False}
    )
```

### 枚举和 JSON 字段

```python
from enum import Enum
from sqlalchemy import Column, JSON

# 使用枚举
class MembershipType(str, Enum):
    FREE = "free"
    MONTHLY = "monthly"

class UserMembership(SQLModel, table=True):
    membership_type: MembershipType = Field(default=MembershipType.FREE)

# 使用 JSON
class Agent(SQLModel, table=True):
    config: Dict[str, Any] = Field(default_factory=dict, sa_column=Column(JSON))
```

## CRUD 操作规范

CRUD 操作应封装在 `api/app/crud/` 目录下的类中。

```python
from sqlmodel import select
from ..models.user import User
from ..schemas.user import UserCreate

class CRUDUser:
    """用户 CRUD 操作类"""

    async def get_by_email(self, db: Session, *, email: str) -> Optional[User]:
        statement = select(User).where(User.email == email, User.is_deleted == False)
        return db.exec(statement).first()

    async def create(self, db: Session, *, obj_in: UserCreate) -> User:
        # ... 创建逻辑，包括密码哈希
        db_obj = User(...)
        db.add(db_obj)
        db.commit()
        db.refresh(db_obj)
        return db_obj

    async def authenticate(self, db: Session, *, email: str, password: str) -> Optional[User]:
        # ... 用户认证逻辑
        pass

user_crud = CRUDUser()
```

## Alembic 数据库迁移

使用 Alembic 管理数据库 schema 变更。

```bash
# 1. 修改 SQLModel 模型后，自动生成迁移脚本
alembic revision --autogenerate -m "描述你的变更"

# 2. 应用迁移到数据库
alembic upgrade head

# 回滚迁移
alembic downgrade -1

# 查看历史
alembic history
```

## 数据库会话管理

通过 FastAPI 的依赖注入系统来管理数据库会话。

```python
# api/app/dependencies/db.py
from ..db.base import engine

def get_db() -> Generator[Session, None, None]:
    """获取数据库会话的依赖"""
    with Session(engine) as session:
        yield session
```

## 查询和事务

- **预加载**: 为避免 N+1 查询问题，在查询关联数据时使用 `selectinload`。

  ```python
  from sqlalchemy.orm import selectinload
  statement = select(User).options(selectinload(User.chats))
  ```

- **事务**: 数据库会话 (`Session`) 自动处理事务。在单个请求中，所有操作都在一个事务内。如果需要精细控制，可以手动调用 `db.commit()`、`db.rollback()` 和 `db.flush()`。

## 最佳实践总结

- **模型设计**: 使用有意义的名称，添加索引，合理使用关联关系。
- **查询优化**: 使用预加载，批量操作，并为常用查询添加缓存。
- **数据安全**: 绝不存储明文密码，使用参数化查询防止 SQL 注入。
- **维护性**: 保持迁移脚本的可追溯性，为模型和字段添加详细注释。

---
> Source: [open-v2ai/build-ai-template](https://github.com/open-v2ai/build-ai-template) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-06 -->
