## vige

> Vige 是一个 FastAPI + Vue 的全栈工程模板，采用前后端分离架构：

# Vige 项目开发规则

## 📋 目录索引
- [项目概述](#项目概述)
- [后端开发规范](#后端开发规范)
- [前端开发规范](#前端开发规范)
- [代码示例模板](#代码示例模板)
- [开发注意事项](#开发注意事项)
- [禁止事项](#禁止事项)
- [推荐做法](#推荐做法)
- [文件命名规范](#文件命名规范)
- [注释规范](#注释规范)

---

## 项目概述
Vige 是一个 FastAPI + Vue 的全栈工程模板，采用前后端分离架构：
- **vige-api**: 后端 API 服务 (FastAPI + SQLAlchemy + PostgreSQL + Redis + Huey)
- **vige-bo**: 后台管理系统 (Vue 2 + iView)
- **vige-web**: 前端用户网站 (Vue 2 + iView)
- **vige-wechat**: 微信 H5 客户端 (Vue 2 + Mint UI)

定位：通用型 Web 框架脚手架，提供认证、任务队列、媒体上传、国际化与统一接口规范，便于二次开发。

## 后端开发规范 (vige-api)

### 技术栈
- Web框架: FastAPI
- ORM: SQLAlchemy 2.0 (新式类型注解)
- 数据库: PostgreSQL + Redis
- 认证: JWT + Cookie + CSRF
- 任务队列: Huey + Redis
- 配置管理: Pydantic Settings

### Pydantic 2.x 兼容性规范 ⚠️ 极其重要

#### 核心变更说明：
项目使用 Pydantic 2.x 版本，某些语法已发生变更，必须严格遵循新版本规范。

#### 字段验证参数变更：
```python
# ❌ 错误：Pydantic 1.x 语法，会导致 PydanticUserError
class UserForm(BaseModel):
    mobile: str = Field(..., description="手机号码", regex="^1[3-9]\d{9}$")

# ✅ 正确：Pydantic 2.x 语法
class UserForm(BaseModel):
    mobile: str = Field(..., description="手机号码", pattern="^1[3-9]\d{9}$")
```

#### 配置类参数变更：
```python
# ❌ 错误：Pydantic 1.x 语法，会产生用户警告
class UserForm(BaseModel):
    username: str = Field(...)
    
    class Config:
        schema_extra = {
            "example": {
                "username": "example"
            }
        }

# ✅ 正确：Pydantic 2.x 语法
class UserForm(BaseModel):
    username: str = Field(...)
    
    class Config:
        json_schema_extra = {
            "example": {
                "username": "example"
            }
        }
```

#### 常见兼容性问题对照表：
```python
# Pydantic 1.x → Pydantic 2.x
regex="pattern"          → pattern="pattern"
schema_extra             → json_schema_extra
allow_reuse=True         → 已移除，无需设置
min_length               → min_length (保持不变)
max_length               → max_length (保持不变)
gt, ge, lt, le          → gt, ge, lt, le (保持不变)
```

#### 验证装饰器使用：
```python
# ✅ 正确：继续使用 field_validator，但注意参数变化
from pydantic import field_validator

class UserForm(BaseModel):
    mobile: str = Field(..., pattern="^1[3-9]\d{9}$")
    
    @field_validator('mobile')
    @classmethod
    def validate_mobile(cls, v):
        if not v:
            raise ValueError('手机号为必填项')
        return v
```

#### 重要注意事项：
1. **严禁使用 `regex`**：必须使用 `pattern` 替代
2. **配置类更新**：使用 `json_schema_extra` 而不是 `schema_extra`
3. **立即修复**：遇到 PydanticUserError 或相关警告立即按规范修复
4. **团队统一**：所有开发者都必须遵循 Pydantic 2.x 语法
5. **检查清单**：每次创建表单类都要检查是否使用了正确的语法

#### 错误信息识别：
- **`PydanticUserError: 'regex' is removed. use 'pattern' instead`** → 将 `regex` 改为 `pattern`
- **`Valid config keys have changed in V2: 'schema_extra' has been renamed to 'json_schema_extra'`** → 将 `schema_extra` 改为 `json_schema_extra`

#### 标准表单模板：
```python
from pydantic import BaseModel, Field
from typing import Optional

class StandardForm(BaseModel):
    """标准表单模板"""
    mobile: str = Field(..., pattern="^1[3-9]\d{9}$")
    username: str = Field(..., min_length=3, max_length=50)
    age: Optional[int] = Field(None, ge=0, le=120)
```

### 代码风格
- 类名: PascalCase (如 User, MediaModel)
- 函数/变量名: snake_case (如 send_validation_code, user_id)
- 常量: UPPER_CASE (如 AUTHJWT_SECRET_KEY)
- 数据库字段: snake_case

### 数据库操作核心原则 ⚠️ 极其重要

#### 基础原则：
1. **只读查询**: 使用 `db: Session = Depends(sm.get_db)` 依赖注入
2. **数据变更**: 使用 `with sm.transaction_scope() as sa:` 事务上下文
3. **用户认证**: 始终使用 `user: User = Depends(get_user)` 进行 JWT 验证
4. **禁止混用**: 不要在同一函数中同时使用两种数据库上下文

#### Session对象管理规范 ⚠️ 极其重要核心原则：

**关键原则：不同session中的对象不能相互操作！**

1. **session隔离原则**：
   - `current_user: User = Depends(get_user)` 获取的对象属于依赖注入的session
   - `with sm.transaction_scope() as sa:` 创建的是新的独立session
   - **绝对禁止**在新session中直接修改其他session的对象

2. **正确的对象修改方式**：
```python
# ❌ 错误：直接修改其他session的对象
@mp_required
async def update_user(form: UpdateForm, current_user: User = Depends(get_user)):
    with sm.transaction_scope() as sa:
        # 错误！current_user属于另一个session，不能在新session中修改
        current_user.nickname = form.nickname  
        current_user.updated_at = datetime.utcnow()

# ✅ 正确：在新session中重新获取对象再修改
@mp_required
async def update_user(form: UpdateForm, current_user: User = Depends(get_user)):
    with sm.transaction_scope() as sa:
        # 正确：在新session中重新获取用户对象
        user = User.get_or_404(sa, current_user.id)
        user.nickname = form.nickname
        user.updated_at = datetime.utcnow()
```

3. **核心要点**：
   - **重新获取对象**：在事务session中必须使用 `Model.get_or_404(sa, id)` 重新获取
   - **ID传递**：可以使用 `current_user.id` 获取ID，但不能直接操作对象
   - **session归属**：每个数据库对象只能在其所属的session中被修改
   - **事务完整性**：确保所有相关操作都在同一个事务session中完成

4. **常见错误模式**：
```python
# ❌ 错误模式1：直接修改依赖注入对象
with sm.transaction_scope() as sa:
    current_user.field = new_value  # 错误！

# ❌ 错误模式2：混用不同session的对象
with sm.transaction_scope() as sa:
    other_user.field = new_value  # 错误！
    current_user.related_id = other_user.id  # 错误！

# ✅ 正确模式：统一在新session中操作
with sm.transaction_scope() as sa:
    user = User.get_or_404(sa, current_user.id)
    other = OtherModel.get_or_404(sa, other_id)
    user.field = new_value
    user.related_id = other.id
```

5. **检查清单**：
   - ✅ 在事务中重新获取需要修改的对象
   - ✅ 使用对象ID进行查询，不直接操作依赖注入的对象
   - ✅ 所有相关对象都在同一个事务session中获取和操作
   - ❌ 不在新session中直接修改current_user等依赖注入对象
   - ❌ 不混用来自不同session的对象

### API 路由设计（与当前项目一致）
- API 基础前缀为 `/v1`
- 端前缀区分: `/admin/*`（后台管理）, `/web/*`（Web 端）, `/wechat/*`（微信端）
- 验证装饰器（项目已有实现）：`@user_required`（仅允许 Web 源）、`@bo_required`（仅允许 BO 源）、`@login_required`（任一受信源）
- 统一响应格式：成功 `{ success: true, data: ... }`；错误 `{ success: false, message: ... }` 或使用 `HTTPException(detail)` 由拦截器展示

### RESTful API 设计规范（保留但按项目实现简化）

#### 核心原则：
严格遵循RESTful API设计规范，使用标准的HTTP方法和资源命名。

#### HTTP方法使用规范：
```python
# ✅ 正确：RESTful API设计
POST   /web/translations           # 创建新的翻译任务
GET    /web/translations           # 获取翻译任务列表
GET    /web/translations/{task_id}  # 获取单个翻译任务详情
PUT    /web/translations/{task_id}  # 更新翻译任务信息
DELETE /web/translations/{task_id}  # 删除翻译任务记录

POST   /web/users          # 创建用户
GET    /web/users          # 获取用户列表  
GET    /web/users/{user_id} # 获取单个用户
PUT    /web/users/{user_id} # 更新用户信息
DELETE /web/users/{user_id} # 删除用户
```

#### 资源命名规范：
1. **使用复数名词**：`/users`、`/asks`、`/products`
2. **避免动词**：不使用 `/search-experts`、`/create-user` 等动词形式
3. **嵌套资源**：`/users/{user_id}/posts`、`/asks/{ask_id}/results`
4. **查询参数**：使用查询参数进行过滤、排序、分页

#### 正确示例：
```python
# ✅ 正确：RESTful设计
@app.post('/web/translations',           # 创建翻译任务
          summary="创建翻译任务")
@app.get('/web/translations',            # 获取翻译列表
         summary="获取翻译历史")
@app.get('/web/translations/{task_id}',   # 获取单个翻译任务
         summary="获取翻译详情")
@app.put('/web/translations/{task_id}',   # 更新翻译任务
         summary="更新翻译信息")
@app.delete('/web/translations/{task_id}', # 删除翻译任务
            summary="删除翻译记录")
```

注：项目部分接口已存在历史路径（如 `/admin/users/<int:user_id>` 风格），逐步演进即可，无需一次性重构。

#### 特殊操作处理：
对于不能用标准CRUD表示的操作，使用子资源或动作：
```python
# 翻译操作作为创建翻译任务的一部分
POST /web/translations  # 创建翻译任务并执行翻译

# 如果需要单独的翻译端点
POST /web/translations/{task_id}/retry  # 重新翻译

# 批量操作
POST /web/translations/batch-delete  # 批量删除
PUT  /web/translations/batch-update   # 批量更新
```

#### RESTful设计检查清单：
- ✅ 使用复数名词作为资源名
- ✅ 使用正确的HTTP方法
- ✅ 返回合适的HTTP状态码
- ✅ 使用查询参数进行过滤
- ✅ 嵌套资源表示关系
- ❌ 不在URL中使用动词
- ❌ 不使用非标准的HTTP方法
- ❌ 不返回错误的状态码

### API 响应格式规范

#### 核心原则：
**所有API接口必须使用统一的 `data` 包装格式，确保响应结构的一致性和扩展性**

#### 标准响应格式：
```python
# ✅ 正确：简单数据返回，使用data包装
return dict(
    success=True,
    data=user.dump()
)

# ✅ 正确：复杂数据返回，data中包含多个字段
return dict(
    success=True,
    data={
        "teachers": [...],
        "pagination": {...},
        "filters": {...},
        "metadata": {...}
    }
)

# ✅ 正确：列表/分页操作返回格式
return dict(
    success=True,
    data={
        "rows": items,
        "total": total_count,
        "page": page,
        "per_page": per_page
    }
)

# ✅ 正确：创建操作返回新对象
return dict(
    success=True,
    data={
        "user": user.dump(),
        "token": access_token
    }
)

# ❌ 错误：字段平铺，不使用data包装
return dict(
    success=True,
    user=user.dump(),          # 错误：应该放在data中
    token=access_token         # 错误：应该放在data中
)

# ❌ 错误：在成功响应中添加不必要的message字段
return dict(
    success=True,
    data=result_data,
    message="操作成功"  # 多余，成功操作不需要message
)
```

#### 使用规范：
1. **统一包装**：所有业务数据必须包装在 `data` 字段中
2. **根级字段**：根级别只保留 `success`、`message`（错误时）等系统字段
3. **成功响应**：只返回 `success=True` 和 `data`，不要添加 `message`
4. **错误响应**：使用标准化错误处理函数，会自动包含合适的错误信息
5. **保持一致性**：所有接口都遵循相同的返回格式
6. **扩展性强**：可以在data中添加任何业务相关字段而不会与系统字段冲突

### 权限验证和来源检查规范 ⭐ 极其重要

#### 核心原则：
不同类型的接口必须使用对应的验证装饰器，确保请求来源的合法性和安全性。

#### 验证装饰器使用规范：
```python
# Web端接口使用 @user_required  
@app.post('/web/translations')
@user_required
async def create_translation():
    pass

# 后台管理接口使用 @bo_required
@app.post('/admin/users')
@bo_required
async def admin_create_user():
    pass

# 微信端接口使用 @wechat_required
@app.post('/wechat/translate')
@wechat_required
async def wechat_translate():
    pass
```

#### 验证装饰器对应关系：
- **Web端接口** (`/web/*`) → `@user_required` - 验证Web端用户身份
- **后台管理接口** (`/admin/*`) → `@bo_required` - 验证后台管理员身份和权限
- **微信端接口** (`/wechat/*`) → `@wechat_required` - 验证微信端用户身份

#### 重要注意事项：
1. **严格对应**：接口前缀必须与验证装饰器严格对应
2. **来源验证**：装饰器会验证请求来源是否合法
3. **用户身份**：同时验证用户的身份和权限
4. **安全性**：防止跨端访问和非法请求

### 标准化错误处理规范 ⭐ 极其重要

#### 核心原则：
1. **统一错误格式**：所有错误都使用标准化的响应格式
2. **标准HTTP状态码**：正确使用HTTP状态码表示不同类型的错误
3. **错误码标识**：为程序化处理提供明确的错误代码
4. **用户友好信息**：提供清晰的错误描述信息

#### 标准错误响应格式：
```python
# 基础错误格式
{
    "success": false,
    "message": "用户友好的错误描述",
    "error_code": "ERROR_CODE",  # 可选
    "details": "详细错误信息"     # 可选
}

# 表单验证错误格式
{
    "success": false,
    "message": "表单验证失败",
    "errors": [
        {
            "field": "username",
            "message": "用户名不能为空"
        }
    ]
}
```

#### HTTP状态码使用规范：
- **200**: 请求成功 - 正常业务操作
- **400**: 请求参数错误 - 参数验证失败、业务逻辑错误
- **401**: 未授权访问 - 未登录、令牌过期、密码错误
- **403**: 权限不足 - 无相应权限、角色限制
- **404**: 资源不存在 - 用户不存在、接口不存在
- **422**: 数据验证错误 - Pydantic表单验证失败
- **500**: 服务器内部错误 - 系统异常、数据库错误

#### 标准错误代码分类：
```python
# 认证相关 (401)
NOT_AUTHENTICATED       # 用户未登录
TOKEN_INVALID          # JWT令牌无效或过期
INVALID_CREDENTIALS    # 用户名或密码错误

# 权限相关 (403)
INSUFFICIENT_PERMISSION # 权限不足
ROLE_RESTRICTION       # 角色权限限制
RESOURCE_FORBIDDEN     # 资源访问限制

# 业务逻辑 (400)
INVALID_PARAMETER      # 请求参数错误
USER_INACTIVE          # 用户未激活
DUPLICATE_RESOURCE     # 资源重复
BUSINESS_RULE_VIOLATION # 业务规则冲突

# 资源相关 (404)
USER_NOT_FOUND         # 用户不存在
RESOURCE_NOT_FOUND     # 资源不存在
API_NOT_FOUND          # API接口不存在

# 系统错误 (500)
INTERNAL_ERROR         # 服务器内部错误
DATABASE_ERROR         # 数据库错误
EXTERNAL_SERVICE_ERROR # 外部服务错误
```

#### 错误处理工具函数使用规范：
```python
# 必须导入和使用标准化错误函数
from ..utils import (
    raise_http_error,           # 通用错误抛出
    raise_bad_request,          # 400错误
    raise_unauthorized,         # 401错误  
    raise_forbidden,            # 403错误
    raise_not_found,            # 404错误
    raise_validation_error,     # 422错误
    raise_internal_error,       # 500错误
    # 业务逻辑快捷函数
    raise_user_not_found,       # 用户不存在
    raise_invalid_credentials,  # 认证失败
    raise_user_inactive,        # 用户未激活
    raise_duplicate_username,   # 用户名重复
    raise_role_not_found,       # 角色不存在
    raise_permission_denied,    # 权限不足
)
```

说明：本项目实际同时使用 `dict(success=..., ...)` 与 `HTTPException` 两种形式，前端拦截器已做兼容；保持一致性即可，后续逐步统一为成功 `{ success, data }`、失败 `{ success=false, message }`。

#### API文档中的错误响应配置：
```python
@router.post('/admin/login')
async def login(...):
    pass
```

#### 自定义错误处理：
```python
# 当需要特殊错误时，使用 raise_http_error
def validate_business_rule(data):
    if some_complex_validation_fails:
        raise_http_error(
            status_code=400,
            message="业务规则验证失败：订单状态不允许此操作",
            error_code="ORDER_STATUS_CONFLICT",
            details={"current_status": "completed", "required_status": "pending"}
        )
```

#### 错误处理原则：
1. **优先使用工具函数**：使用预定义的错误函数而不是直接抛HTTPException
2. **正确的状态码**：确保HTTP状态码与错误类型匹配
3. **清晰的错误信息**：提供用户友好的中文错误描述
4. **错误码一致性**：相同类型的错误使用相同的错误代码
5. **文档完整性**：API文档中必须包含错误响应示例
6. **日志记录**：重要错误需要记录详细的日志信息

#### 错误处理检查清单：
- ✅ 使用标准化错误函数
- ✅ HTTP状态码正确
- ✅ 包含 STANDARD_ERROR_RESPONSES
- ✅ 错误信息用户友好
- ✅ 错误码符合规范
- ❌ 不使用直接的 HTTPException
- ❌ 不使用自定义错误格式
- ❌ 不混用状态码



### 表单验证和参数处理规范（结合项目现状）

#### 核心原则：
优先使用 Pydantic 2.x 表单进行验证与文档生成；GET 场景可视复杂度选择 `Depends()` 表单或简洁的 Query 参数，保持易读与一致性。

#### 建议：
- POST 接口使用表单模型接收请求体
- GET 接口优先表单依赖；若参数极少可直接使用 Query 参数
- 参数集中管理，便于统一验证与生成文档

（移除与本项目无关的 `mp` 示例）

（移除与本项目无关的反例段落）

#### 表单设计原则：
- **所有API接口都必须使用表单验证**
- **Filter Form**：用于查询和分页，继承 `BaseFilterForm`
- **Edit Form**：用于创建和编辑，Create 和 Update 可以使用同一个表单
- **参数集中**：相关参数组织到同一个表单类中
- **类型安全**：利用 Pydantic 提供类型检查和自动转换
- **文档完整**：表单字段自动生成完整的 API 文档

#### Filter Form 规范：
```python
from ..forms import BaseFilterForm

class GenerationRequestFilterForm(BaseFilterForm):
    product_id: Optional[int] = None
    status: Optional[str] = None
    
    # BaseFilterForm 已包含：
    # keyword: Optional[str] = None  # 关键词过滤
    # page: Optional[int] = 1        # 页码
    # per_page: Optional[int] = 10   # 每页数量
```

#### Edit Form 规范：
```python
class GenerationRequestForm(BaseModel):
    product_id: int
    prompt: str
    size: str = OpenAIImageSize.SQUARE.value
    style: str = OpenAIImageStyle.VIVID.value
    image_count: int = 1
```

#### 分页查询使用 CRUDMixin：
```python
@router.get('/web/ai/generation-history')
@user_required
async def get_generation_history(
    form: GenerationRequestFilterForm = Depends(),
    db: Session = Depends(sm.get_db),
    current_user: User = Depends(get_user)
):
    query = db.query(GenerationRequest).filter(
        GenerationRequest.user_id == current_user.id
    )
    
    # keyword 过滤主要字段
    if form.keyword:
        query = query.filter(
            GenerationRequest.prompt.ilike(f'%{form.keyword}%')
        )
    
    # 其他过滤条件
    if form.product_id:
        query = query.filter(GenerationRequest.product_id == form.product_id)
    
    # 使用 CRUDMixin 的分页方法
    return GenerationRequest.paginated_dump(
        query=query,
        form=form,
        dump_func=lambda req: {
            **req.dump(),
            'generated_images': [img.dump() for img in req.generated_images]
        }
    )
```

#### 创建/编辑使用表单：
```python
@router.post('/web/ai/generate-images')
@user_required
async def generate_images(
    form: GenerationRequestForm,
    current_user: User = Depends(get_user)
):
    # 使用 form.dict() 获取验证后的数据
    with sm.transaction_scope() as sa:
        generation_request = GenerationRequest.create(
            sa,
            user_id=current_user.id,
            **form.dict()
        )
```

#### 关键词过滤规范：
- **必须支持 keyword 过滤**：所有 Filter Form 都要处理 keyword 字段
- **使用 ilike 模糊匹配**：`field.ilike(f'%{keyword}%')`
- **过滤主要字段**：选择最重要的可搜索字段（如名称、描述等）

#### 表单验证原则：
1. **输入验证**：所有用户输入都通过 Pydantic 表单验证
2. **类型安全**：利用 Pydantic 的类型检查和转换
3. **默认值**：为可选字段提供合理的默认值
4. **复用性**：Create 和 Update 可以使用同一个表单
5. **分离关注点**：Filter 和 Edit 表单职责分离

### MediaModel 使用规范（与当前实现一致）

#### 核心设计原则：
**MediaModel 是独立的文件存储表，不主动关联其他业务表，其他表通过外键引用 MediaModel**

#### MediaModel 表结构特点：
```python
class MediaModel(CRUDMixin, ProfileMixin):
    __tablename__ = 'media'
    
    # 通用关联字段，用于关联任意业务对象
    object_type: Mapped[str] = mapped_column(Unicode, nullable=True)
    object_id: Mapped[int] = mapped_column(BigInteger, nullable=True)
    
    @declared_attr
    def object(cls):
        return generic_relationship(cls.object_type, cls.object_id)
```

#### 关联关系设计原则：
1. **MediaModel 独立存在**：不包含 user_id、product_id 等业务相关外键
2. **业务表主动引用**：其他表通过外键引用 MediaModel.id
3. **通用关联机制**：通过 object_type 和 object_id 实现通用关联

#### 正确的关联方式：
```python
# ✅ 正确：业务表主动引用 MediaModel
class Product(CRUDMixin, ProfileMixin, TrackableMixin):
    __tablename__ = 'products'
    
    user_id: Mapped[int] = mapped_column(BigInteger, ForeignKey('users.id'))
    original_image_id: Mapped[int] = mapped_column(BigInteger, ForeignKey('media.id'))
    
class User(CRUDMixin, ProfileMixin, TrackableMixin):
    __tablename__ = 'users'
    
    avatar_id: Mapped[Optional[int]] = mapped_column(BigInteger, ForeignKey('media.id'), nullable=True)
```

#### 正确的查询方式：
```python
# ✅ 正确：只验证 MediaModel 是否存在
@router.post('/web/products')
@user_required
async def create_product(
    form: ProductForm,
    current_user: User = Depends(get_user)
):
    with sm.transaction_scope() as sa:
        # 只查询 media 是否存在，不添加用户关联条件
        media = sa.query(MediaModel).filter(
            MediaModel.id == form.original_image_id
        ).first()
        
        if not media:
            raise_not_found("图片不存在")
        
        # 在业务对象中建立关联关系
        product = Product.create(
            sa,
            user_id=current_user.id,
            original_image_id=form.original_image_id,
            **form.dict(exclude={'original_image_id'})
        )
        return dict(success=True, data=product.dump())
```

说明：`MediaModel` 仅维护 `object_type/object_id` 通用关联与 `profile`。不要为其添加业务外键。

#### 权限验证的正确方式：
如果需要验证用户权限，应该通过业务逻辑处理：
```python
# ✅ 正确：通过业务表验证权限
def verify_media_ownership(sa: Session, media_id: int, user_id: int, table_class):
    """验证用户是否拥有指定的媒体文件（通过业务表）"""
    return sa.query(table_class).filter(
        table_class.original_image_id == media_id,
        table_class.user_id == user_id
    ).first() is not None
```

### 文件上传规范 ⭐ 重要

项目使用统一的文件上传机制，所有文件上传都通过 media 模块处理：

#### 统一上传接口：
- **接口地址**：`POST /media`
- **认证要求**：使用 `@login_required` 装饰器
- **返回数据**：MediaModel 对象，包含 id、filename、url 等信息
- **支持格式**：png, jpg, jpeg, gif（在 media/forms.py 中定义）

#### 文件上传流程：
1. **前端上传**：调用 `POST /media` 接口上传文件
2. **获取 media_id**：从响应中获取 MediaModel 的 id
3. **业务关联**：在业务接口中使用 media_id 关联到具体业务对象

#### 错误做法（禁止）：
```python
# ❌ 错误：不要在业务模块中重复实现文件上传
@router.post('/web/products/upload')
async def upload_product(file: UploadFile = File(...)):
    # 错误：重复实现文件上传逻辑
    pass
```

#### 文件上传规范：
1. **统一接口**：所有文件上传都使用 `/media` 接口
2. **职责分离**：media 模块负责文件处理，业务模块负责业务逻辑
3. **独立验证**：只验证 MediaModel 是否存在，不添加用户关联查询
4. **类型检查**：在业务逻辑中可以额外验证文件类型是否符合业务需求
5. **错误处理**：统一的文件格式和大小限制在 media 模块中处理

### 数据模型设计
- 继承 Mixin: CRUDMixin, ProfileMixin, TrackableMixin, SoftDeleteMixin
- 使用 SQLAlchemy 2.0 语法: `Mapped[type] = mapped_column(...)`
- JSON 扩展字段使用 ProfileMixin 的属性装饰器

### ProfileMixin 使用规范 ⭐ 核心重要

ProfileMixin 提供了灵活的 JSONB 字段扩展机制，是项目的核心设计模式：

#### ProfileMixin 对象创建规范 ⚠️ 极其重要

**核心问题**：在创建继承了 ProfileMixin 的对象时，**绝对不能在 create() 方法中直接传入通过属性装饰器定义的字段**，这会导致 `'NoneType' object does not support item assignment` 错误。

#### 正确的对象创建方式：
```python
# ✅ 正确：分两步创建对象
with sm.transaction_scope() as sa:
    # 第一步：只传入直接的数据库字段
    task = TranslationTask.create(
        sa,
        user_id=current_user.id,
        task_no=task_no,
        title=form.title,
        translation_type=form.translation_type,
        source=form.source,  # 直接数据库字段
        source_language=form.source_language,
        target_language=form.target_language,
        ai_model=form.ai_model,
        status=TranslationStatus.PROCESSING,
        started_at=datetime.utcnow()
    )
    
    # 第二步：单独设置 ProfileMixin 属性
    task.source_text = form.text  # @string_property
    task.ip_address = request.client.host  # @string_property
    task.translation_config = {"url": form.url}  # @object_property
```

#### 错误做法（严禁）：
```python
# ❌ 错误：直接在create()中传入ProfileMixin属性
task = TranslationTask.create(
    sa,
    user_id=current_user.id,
    source_text=form.text,  # 错误！这是@string_property，不是直接字段
    ip_address=request.client.host,  # 错误！这是@string_property
    translation_config={"url": form.url}  # 错误！这是@object_property
)
```

#### 原因说明：
1. **属性装饰器依赖**：`@string_property`、`@object_property` 等装饰器需要 `profile` 字段已经初始化
2. **初始化顺序**：在 `create()` 过程中，`profile` 字段可能还没有正确初始化为空字典
3. **赋值失败**：当装饰器尝试设置 `instance.profile[self.name] = value` 时，如果 `profile` 为 None，就会出现 NoneType 错误

#### 识别 ProfileMixin 属性的方法：
- 查看模型定义中使用了 `@string_property`、`@integer_property`、`@object_property`、`@array_property` 等装饰器的字段
- 这些字段存储在 JSONB 的 `profile` 字段中，不是直接的数据库列

#### 检查清单：
- ✅ 创建对象时只传入直接的数据库字段（有 `mapped_column` 定义的字段）
- ✅ 对象创建后，单独设置 ProfileMixin 属性
- ✅ 区分直接字段和 ProfileMixin 属性
- ❌ 不要在 create() 中传入任何使用属性装饰器定义的字段
- ❌ 不要忽视 `'NoneType' object does not support item assignment` 错误

#### 字段分类原则：
**核心字段（直接定义为数据库列）**：
- 需要索引查询的字段（如 user_id, status, created_at）
- 需要外键关联的字段（如 user_id, image_id）
- 需要数据库约束的字段（如 unique, not null）
- 需要排序或过滤的字段
- 核心业务逻辑字段

**扩展字段（使用 ProfileMixin 的 profile JSONB 字段）**：
- 可选的元数据字段（如文件大小、原始文件名）
- 不需要索引的描述性字段（如备注、标签）
- 可能变化的业务字段（如用户偏好设置）
- 复杂数据类型字段（如数组、对象）
- 第三方服务返回的额外信息

#### 可用的属性装饰器：
```python
@string_property        # 字符串字段
@integer_property       # 整数字段  
@float_property         # 浮点数字段
@bool_property          # 布尔字段
@array_property         # 数组字段
@object_property        # 对象字段
@date_property          # 日期字段
@datetime_property      # 日期时间字段
@price_property         # 价格字段（自动处理分/元转换）
```

#### 标准使用模式：
```python
class Product(CRUDMixin, ProfileMixin, TrackableMixin):
    __tablename__ = 'products'
    
    # 核心字段 - 直接定义为数据库列
    user_id: Mapped[int] = mapped_column(BigInteger, ForeignKey('users.id'), index=True)
    original_image_id: Mapped[int] = mapped_column(BigInteger, ForeignKey('media.id'))
    cutout_status: Mapped[str] = mapped_column(Unicode(50), default='pending', index=True)
    
    # 扩展字段 - 使用装饰器存储在 profile JSONB 中
    @string_property
    def original_filename(self):
        """原始文件名"""
        pass
    
    @integer_property
    def file_size(self):
        """文件大小（字节）"""
        pass
    
    @array_property
    def tags(self):
        """标签列表"""
        pass
    
    @object_property
    def metadata(self):
        """图片元数据（宽高、格式等）"""
        pass
    
    def dump(self):
        return dict(
            id=self.id,
            user_id=self.user_id,
            cutout_status=self.cutout_status,
            # 扩展字段可以直接访问
            original_filename=self.original_filename,
            file_size=self.file_size,
            tags=self.tags,
            metadata=self.metadata,
            created_at=self.created_at.strftime(DATETIME_FORMAT),
        )
```

#### 重要注意事项：
1. **装饰器函数体必须是 `pass`**，不要写其他代码
2. **扩展字段访问就像普通属性**，无需特殊语法
3. **类型转换自动处理**，装饰器会自动转换数据类型
4. **默认值处理**：如果字段不存在，装饰器会调用函数名对应的默认值工厂
5. **修改扩展字段会自动标记 profile 字段为已修改**，确保数据库更新

#### 错误示例（禁止）：
```python
# ❌ 错误：不要在装饰器函数中写逻辑
@string_property
def brand(self):
    return "default_brand"  # 错误！应该是 pass

# ❌ 错误：需要索引的字段不应该放在 profile 中
@string_property
def user_id(self):  # 错误！user_id 需要索引，应该是直接字段
    pass
```

#### 正确示例：
```python
# ✅ 正确：扩展字段使用装饰器
@string_property
def brand(self):
    """品牌名称"""
    pass

# ✅ 正确：核心字段直接定义
user_id: Mapped[int] = mapped_column(BigInteger, ForeignKey('users.id'), index=True)
```

### 模块结构
每个功能模块包含:
- api.py: 路由和控制器
- models.py: 数据模型
- forms.py: 请求验证 (Pydantic)
- constants.py: 模块常量

#### 文件职责分离 ⭐ 重要
- **forms.py**: 包含请求表单验证模型，用于验证用户输入
- **api.py**: 路由定义和业务逻辑实现，在responses中提供响应示例
- **models.py**: 数据库模型定义，与API层分离
- **constants.py**: 枚举常量和业务规则定义

### 常量定义规范 ⭐ 重要

使用 Enum 和 IntEnum 定义模块常量，提供类型安全和扩展属性：

#### 基础枚举定义：
```python
from ..utils import Enum, IntEnum
from ..fixtures import register_enum

class CutoutStatus(IntEnum):
    PENDING = 10
    PROCESSING = 20
    COMPLETED = 30
    FAILED = 40

# 设置标签
CutoutStatus.PENDING.label = '等待抠图'
CutoutStatus.PROCESSING.label = '抠图中'
CutoutStatus.COMPLETED.label = '抠图完成'
CutoutStatus.FAILED.label = '抠图失败'

# 注册枚举（用于fixtures）
register_enum(CutoutStatus)
```

#### 带扩展属性的枚举：
```python
class ConversationMode(IntEnum):
    creative = 10
    balance = 20
    accuracy = 30

ConversationMode.creative.label = '更多创造力'
ConversationMode.balance.label = '更多平衡'
ConversationMode.accuracy.label = '更多精确'

# 扩展属性
ConversationMode.creative.temperature = 1
ConversationMode.balance.temperature = 0.5
ConversationMode.accuracy.temperature = 0.1

# 如果有扩展属性，重写dump方法
def dump(self):
    return dict(
        name=self.name,
        label=self.label,
        value=self.value,
        temperature=self.temperature,
    )
ConversationMode.dump = dump

register_enum(ConversationMode)
```

#### 常量定义原则：
1. **使用 IntEnum**：当需要数据库存储和比较时
2. **使用 Enum**：当只需要字符串标识时
3. **设置 label**：提供用户友好的显示文本
4. **扩展属性**：为枚举添加业务相关的配置
5. **注册枚举**：使用 register_enum 注册到 fixtures 系统
6. **重写 dump**：当有扩展属性时，重写 dump 方法包含所有属性

### 枚举常量统一返回规范 ⭐ 重要

项目使用统一的 fixtures 机制来返回所有枚举常量：

#### Fixtures 机制：
- **统一接口**：所有枚举常量通过 `/admin/fixtures` 接口统一获取
- **自动注册**：使用 `register_enum(EnumClass)` 自动注册到 fixtures 系统
- **前端使用**：前端通过一次请求获取所有枚举常量，无需单独接口

#### 正确做法：
```python
# constants.py - 定义枚举并注册
class CutoutStatus(IntEnum):
    PENDING = 10
    COMPLETED = 30

CutoutStatus.PENDING.label = '等待抠图'
CutoutStatus.COMPLETED.label = '抠图完成'

register_enum(CutoutStatus)  # 注册到fixtures系统
```

#### 错误做法（禁止）：
```python
# ❌ 错误：不要为枚举常量单独创建API接口
@router.get('/web/cutout-status-options')
def get_cutout_status_options():
    # ❌ 错误：应该使用统一的fixtures机制
return [status.dump() for status in CutoutStatus]
```

#### 使用原则：
1. **统一获取**：枚举常量通过 fixtures 统一返回，不要重复创建接口
2. **自动注册**：所有枚举都必须使用 `register_enum()` 注册
3. **前端缓存**：前端可以缓存 fixtures 数据，减少重复请求
4. **保持一致**：所有模块的枚举常量都遵循相同的获取方式

### 配置信息与常量分离规范 ⭐ 重要

严格区分配置信息和常量定义，确保代码的可维护性和安全性：

#### 配置信息（config.py）：
- **API密钥和令牌**：如 `REMOVE_BG_API_KEY`、`GPT_TOKEN`
- **服务端点和URL**：如 `EXTERNAL_URL`、`REDIS_HOST`
- **模型名称和版本**：如 `GPT_MODAL_NAME`、`GPT_ADVANCED_MODAL_NAME`
- **数量限制和阈值**：如 `GPT_MAX_OUTPUT_TOKENS`、`AI_GENERATE_WORD_LIMIT`
- **超时时间和重试次数**：如具体的数值配置
- **文件大小限制**：如 `DEFAULT_THUMBNAIL_SIZE`
- **环境相关配置**：如 `DEBUG`、`ENV`

#### 常量定义（constants.py）：
- **枚举类型**：如状态枚举、类型枚举
- **固定的业务规则**：如支持的文件格式列表
- **不变的标识符**：如错误代码、事件类型

#### 正确示例：
```python
# config.py - 配置信息
class Settings(BaseSettings):
    REMOVE_BG_API_KEY: str = 'your_api_key'
    REMOVE_BG_TIMEOUT: int = 30
    REMOVE_BG_MAX_RETRIES: int = 3
    OPENAI_MODEL_NAME: str = 'dall-e-3'
    OPENAI_MAX_IMAGES: int = 4

# constants.py - 枚举常量
class AIServiceType(IntEnum):
    REMOVE_BG = 10
    OPENAI_DALLE = 20

class AIServiceStatus(IntEnum):
    PENDING = 10
    SUCCESS = 30
    FAILED = 40

# 固定的业务规则
SUPPORTED_IMAGE_FORMATS = ['jpg', 'jpeg', 'png', 'webp']
```

#### 错误示例（禁止）：
```python
# ❌ 错误：不要在 constants.py 中放配置信息
REMOVE_BG_API_KEY = 'your_api_key'  # 应该在 config.py
OPENAI_TIMEOUT = 60  # 应该在 config.py

# ❌ 错误：不要在 config.py 中定义枚举
class AIServiceType(IntEnum):  # 应该在 constants.py
    REMOVE_BG = 10
```

#### 使用原则：
1. **配置可变**：放在 config.py，支持环境变量覆盖
2. **常量不变**：放在 constants.py，代码中直接引用
3. **安全考虑**：敏感信息（API密钥）必须在 config.py 中
4. **部署灵活**：配置信息支持不同环境的不同值
5. **代码清晰**：常量定义提供类型安全和标签

### 枚举类型在数据模型中的使用规范（可选）

当在数据模型中使用枚举类型时，遵循以下规范：

#### 数据库字段定义：
```python
from .constants import CutoutStatus

class Product(CRUDMixin, ProfileMixin, TrackableMixin):
    __tablename__ = 'products'
    
    # 使用 Mapped[EnumType] 类型注解，数据库存储为 Integer
    cutout_status: Mapped[CutoutStatus] = mapped_column(
        Integer, nullable=False, default=CutoutStatus.PENDING, 
        index=True, comment='抠图状态'
    )
```

说明：项目中暂未大量使用 `IntEnum` 直接映射到列（但工具已提供），可按需采用并在 `dump()` 中进行初始化再输出。

#### 枚举字段使用原则：
1. **类型注解**：使用 `Mapped[EnumType]` 而不是 `Mapped[int]`
2. **数据库类型**：mapped_column 中使用 `Integer` 存储枚举值
3. **默认值**：直接使用枚举成员作为默认值（如 `CutoutStatus.PENDING`）
4. **序列化**：在 dump 方法中调用 `EnumType.init(self.field_name).dump()` 获取完整枚举信息
5. **索引**：状态类枚举字段通常需要添加索引用于查询过滤

#### 完整示例：
```python
# constants.py
class CutoutStatus(IntEnum):
    PENDING = 10
    PROCESSING = 20
    COMPLETED = 30
    FAILED = 40

CutoutStatus.PENDING.label = '等待抠图'
CutoutStatus.PROCESSING.label = '抠图中'
CutoutStatus.COMPLETED.label = '抠图完成'
CutoutStatus.FAILED.label = '抠图失败'

register_enum(CutoutStatus)

# models.py
class Product(CRUDMixin, ProfileMixin, TrackableMixin):
    __tablename__ = 'products'
    
    cutout_status: Mapped[CutoutStatus] = mapped_column(
        Integer, nullable=False, default=CutoutStatus.PENDING, 
        index=True, comment='抠图状态'
    )
    
    def dump(self):
        return dict(
            id=self.id,
            cutout_status=CutoutStatus.init(self.cutout_status).dump(),  # 返回 {name, label, value}
            created_at=self.created_at.strftime(DATETIME_FORMAT),
        )
```

#### 注意事项：
- ✅ 使用 `Mapped[EnumType]` 类型注解提供类型安全
- ✅ 数据库字段使用 `Integer` 类型存储枚举值
- ✅ dump 方法中调用 `EnumType.init(field).dump()` 获取枚举的完整信息
- ❌ 不要使用 `Mapped[int]` 丢失类型信息
- ❌ 不要在 dump 中直接返回枚举值，应该先调用 `EnumType.init()` 再调用 `.dump()` 方法

### 数据模型dump方法规范 ⭐ 极其重要

#### 核心原则：
所有数据模型的dump方法必须正确处理枚举字段和时间字段，确保返回格式的一致性。

#### 枚举字段处理规范：
**关键问题**：数据库存储的是int类型，直接调用 `.dump()` 会报错，必须先初始化为枚举对象。

```python
# ❌ 错误：直接调用dump会报错，因为是int类型
def dump(self):
    return dict(
        status=self.status.dump(),  # 错误！self.status是int，没有dump方法
        degree_level=self.degree_level.dump()  # 错误！
    )

# ✅ 正确：先用EnumType.init()初始化，再调用dump()
def dump(self):
    return dict(
        status=EducationStatus.init(self.status).dump(),
        degree_level=DegreeLevel.init(self.degree_level).dump(),
    )
```

#### 时间字段格式化规范：
当前项目未提供统一的时区/格式化工具函数。可以在各模型中按需进行格式化；若后续新增统一工具，再逐步迁移统一写法。

#### 标准dump方法模板：
```python
from .constants import StatusEnum, TypeEnum

class ExampleModel(CRUDMixin, ProfileMixin, TrackableMixin):
    __tablename__ = 'examples'
    
    status: Mapped[StatusEnum] = mapped_column(Integer, default=StatusEnum.PENDING)
    type: Mapped[TypeEnum] = mapped_column(Integer, default=TypeEnum.DEFAULT)
    
    def dump(self):
        """序列化输出"""
        return dict(
            id=self.id,
            # 枚举字段：先init再dump
            status=StatusEnum.init(self.status).dump(),
            type=TypeEnum.init(self.type).dump(),
            
            # 时间字段：可按需格式化，或在后续引入统一函数后统一替换
            # created_at=format_utc_to_shanghai_str(self.created_at),
            # updated_at=format_utc_to_shanghai_str(self.updated_at),
            
            # 其他字段
            name=self.name,
            description=self.description,
        )
```

#### dump方法开发检查清单：
- ✅ 枚举字段使用 `EnumType.init(self.field).dump()` 格式
- （建议）如后续提供统一函数，再统一替换为该函数
- ❌ 不要直接调用枚举的dump方法：`self.status.dump()`
- ❌ 不要使用 strftime(DATETIME_FORMAT)：`self.created_at.strftime(DATETIME_FORMAT)`
- ❌ 不要使用isoformat()：`self.created_at.isoformat()`
- ❌ 不要手动添加 None 检查：`self.updated_at.strftime(...) if self.updated_at else None`

#### 常见错误修复对照表：
```python
# 错误 → 正确
self.status.dump() → StatusEnum.init(self.status).dump()
self.created_at.strftime(DATETIME_FORMAT) → format_utc_to_shanghai_str(self.created_at)
self.created_at.isoformat() → format_utc_to_shanghai_str(self.created_at)
self.updated_at.strftime(DATETIME_FORMAT) if self.updated_at else None → format_utc_to_shanghai_str(self.updated_at)
```

## 前端开发规范 (vige-web, vige-bo, vige-wechat)

### 技术栈选择
- vige-bo (后台): Vue 2 + iView (功能丰富的管理界面)
- vige-web (前端): Vue 2 + iView (保持一致性)
- vige-wechat (微信): Vue 2 + Mint UI (移动端优化)

### 共同技术栈
- Vue Router (路由管理)
- Vuex (状态管理)
- vue-auth (认证管理)
- axios (HTTP 客户端)
- vue-i18n (国际化)
- Less (CSS 预处理器)

### API 响应处理规范 ⚠️ 极其重要

#### axios 响应拦截器自动处理
项目中的 `api.js` 已配置响应拦截器，**自动提取 data 字段**：

```javascript
api.interceptors.response.use(response => {
  if (!response.config.hideProgress) {
    iView.LoadingBar.finish()
  }
  return response.data  // 自动返回 response.data
}, (error) => {
```

#### 正确的API响应处理方式：
```javascript
// ✅ 正确：直接使用 response，不需要 .data
const response = await this.$http.post('/web/ai/cutout-image', data)

if (response && response.success) {
  // 直接访问 response.data，不是 response.data.data
  this.result = response.data
  this.$Message.success(response.message)
} else {
  this.$Message.error(response.message || '操作失败')
}
```

#### 错误的处理方式（禁止）：
```javascript
// ❌ 错误：重复提取 data 字段
const response = await this.$http.post('/web/api', data)

if (response.data && response.data.success) {  // 错误！
  this.result = response.data.data  // 错误！
}
```

#### API响应结构说明：
- **后端API返回**：`{success: true, data: {...}, message: "..."}`
- **axios拦截器处理后**：直接返回上述结构，无需再次访问 `.data`
- **前端使用**：`response.success`、`response.data`、`response.message`

### 接口调用规范 ⚠️ 重要

#### 核心原则：
1. **API URL 规范**: api.js 中已设置 baseURL 为 '/v1'，前端调用时不要重复添加 '/v1' 前缀
2. **接口确认**: 如果不确定应该调用哪个接口，必须先询问确认，不要猜测或调用错误的接口
3. **统一错误处理**：所有接口调用都要有适当的错误处理

#### 正确的接口调用方式：
```javascript
// ✅ 正确：Web端接口使用 /web/ 前缀
const response = await this.$http.post('/web/translations', data)
const response = await this.$http.get('/web/users/123')

// ✅ 正确：后台管理接口使用 /admin/ 前缀  
const response = await this.$http.post('/admin/users', data)
const response = await this.$http.get('/admin/settings')

// ❌ 错误：使用了错误的前缀
const response = await this.$http.post('/api/users', data)  // 应该是 /web/ 或 /admin/
const response = await this.$http.post('/v1/web/users', data)  // 重复的 /v1 前缀
```

#### 接口调用流程：
1. **确认接口**: 先确认要调用的正确接口路径和参数
2. **使用正确URL**: 使用相对路径，不包含 /v1 前缀
3. **错误处理**: 添加 try-catch 和用户友好的错误提示
4. **调试信息**: 在开发模式下添加 console.log 用于调试

#### 错误做法（禁止）：
```javascript
// ❌ 错误：调用不存在的接口
async fetchData() {
  const response = await this.$http.get('/web/some/nonexistent/api')
}

// ❌ 错误：没有错误处理
async fetchData() {
  const response = await this.$http.get('/web/users/me')
  this.data = response.data // 可能导致运行时错误
}

// ❌ 错误：重复的 /v1 前缀
async fetchData() {
  const response = await this.$http.get('/v1/web/users/me') // 会变成 /v1/v1/web/users/me
}
```

### ES语法兼容性规范 ⚠️ 极其重要

#### 禁用的ES2020+语法：
项目使用较旧的Babel配置，**严禁使用**以下ES2020+语法：

1. **可选链操作符 `?.`**：
```javascript
// ❌ 错误：会导致编译错误
const value = obj?.property?.subProperty
const result = func?.()
const item = arr?.[0]

// ✅ 正确：使用传统的安全检查
const value = obj && obj.property && obj.property.subProperty
const result = func && func()
const item = arr && arr[0]
```

2. **空值合并操作符 `??`**：
```javascript
// ❌ 错误：会导致编译错误
const value = input ?? defaultValue

// ✅ 正确：使用传统的默认值检查
const value = input !== null && input !== undefined ? input : defaultValue
// 或者对于falsy值：
const value = input || defaultValue
```

3. **动态导入 `import()`**：
```javascript
// ❌ 错误：可能不被支持
const module = await import('./module.js')

// ✅ 正确：使用静态导入或require
import module from './module.js'
// 或者在需要时使用webpack的require.ensure
```

#### 安全的数据访问模式：
```javascript
// ✅ 正确：完整的链式检查
if (response && response.data && response.data.user) {
  this.user = response.data.user
}

// ✅ 正确：使用逻辑运算符
const userName = user && user.profile && user.profile.name || '未知用户'

// ✅ 正确：三元运算符
const isValid = data && data.status ? data.status === 'active' : false

// ✅ 正确：提前返回模式
if (!response || !response.data) {
  return
}
// 安全使用 response.data
```

#### 编译错误预防原则：
1. **避免现代语法**：不使用ES2020+的新语法特性
2. **传统检查**：使用 `&&` 运算符进行安全的属性访问
3. **兼容性优先**：优先考虑代码的兼容性而不是简洁性
4. **测试编译**：修改代码后及时检查是否有编译错误
5. **渐进增强**：如果需要现代语法，确保先升级Babel配置

#### 常见错误修复对照表：
```javascript
// 错误 → 正确
obj?.prop → obj && obj.prop
obj?.method?.() → obj && obj.method && obj.method()
arr?.[index] → arr && arr[index]
value ?? default → value !== null && value !== undefined ? value : default
response?.data?.success → response && response.data && response.data.success
this.uploadedFile?.original_filename → this.uploadedFile && this.uploadedFile.original_filename
```

#### 实际错误案例：
**错误代码**：
```javascript
// ❌ 导致编译错误：Unexpected token (491:47)
const originalName = this.uploadedFile?.original_filename || 'image';
```

**错误信息**：
```
Module parse failed: Unexpected token (491:47)
You may need an additional loader to handle the result of these loaders.
```

**正确修复**：
```javascript
// ✅ 正确：使用传统安全检查
const originalName = this.uploadedFile && this.uploadedFile.original_filename || 'image';
```

### 图片上传逻辑规范 ⭐ 重要

#### 核心概念：
用户上传图片会创建图片对象（MediaModel），并返回 media ID，这个 ID 才是我们需要写入指定表的信息，以绑定图片实例。

#### 标准流程：
1. **上传图片**：调用 `POST /media` 接口上传文件
2. **获取 media_id**：从响应中获取 MediaModel 的 id
3. **绑定业务对象**：使用 media_id 创建或更新业务记录

#### 正确实现：
```javascript
// 步骤1：上传图片到 media 接口
async handleUploadSuccess(response) {
  if (response.success) {
    const mediaId = response.data.id // 这是 media ID
    
    // 步骤2：使用 media ID 创建业务对象
    await this.createProduct(mediaId)
  }
}

// 步骤3：创建产品记录，传入 media ID
async createProduct(mediaId) {
  // TODO: 调用产品创建接口
  // POST /web/products { original_image_id: mediaId }
  
  const response = await this.$http.post('/web/products', {
    original_image_id: mediaId,
    product_name: this.productName
  })
}
```

#### 错误理解（禁止）：
```javascript
// ❌ 错误：直接使用图片URL而不是media ID
async createProduct() {
  const response = await this.$http.post('/web/products', {
    image_url: this.imageUrl // 错误：应该使用 media_id
  })
}

// ❌ 错误：跳过 media 上传，直接上传到业务接口
async uploadProductImage(file) {
  // 错误：应该先上传到 /media 获取 ID
  const response = await this.$http.post('/web/products/upload', {
    file: file
  })
}
```

### Vue 组件结构
```javascript
export default {
  name: 'ComponentName',
  props: {},
  data() {
    return {}
  },
  computed: {},
  methods: {}
}
```

### API 调用风格
- 使用 `this.$http` (axios)
- 处理响应的 success 字段
- 统一错误处理

### 权限控制
- 使用 Mixin 方式: `this.hasPermission('permission.name')`
- 路由懒加载: `component: () => import('@/views/...')`

### 用户认证
- 使用 vue-auth 获取用户信息：`this.$auth.user()`
- 检查登录状态：`this.$auth.check()`
- 避免不必要的用户信息接口调用

## 代码示例模板

### 后端 API 示例

#### 查询操作
```python
from fastapi import Path, Query
from ..forms import STANDARD_ERROR_RESPONSES
from ..utils import raise_user_not_found

@app.get('/web/users/{user_id}')
@user_required
async def get_user(
    user_id: int = Path(..., description="用户ID", example=1, gt=0),
    db: Session = Depends(sm.get_db),
    current_user: User = Depends(get_user)
):
    user = db.query(User).filter(User.id == user_id).first()
    if not user:
        raise_user_not_found()
    return dict(success=True, data=user.dump())
```

#### 数据变更操作
```python
from ..utils import raise_duplicate_username, raise_role_not_found

class CreateUserForm(BaseModel):
    """创建用户表单"""
    username: str = Field(
        min_length=3,
        max_length=50,
        pattern="^[a-zA-Z0-9_]+$"
    )
    password: str = Field(
        min_length=8
    )
    nickname: str = Field(
        max_length=100
    )

@app.post('/web/users')
@user_required
async def create_user(
    form: CreateUserForm,
    current_user: User = Depends(get_user)
):
    # 检查用户名是否重复
    if User.exists(db, User.username == form.username):
        raise_duplicate_username()
    
    with sm.transaction_scope() as sa:
        user = User.create(sa, **form.dict())
        user.created_by = current_user.id
    
    return dict(success=True, data=user.dump())
```

### 前端 Vue 组件示例
```vue
<template>
  <div class="component-name">
    <i-table :data="tableData" :columns="columns"></i-table>
  </div>
</template>

<script>
export default {
  name: 'ComponentName',
  data() {
    return {
      tableData: [],
      columns: []
    }
  },
  mounted() {
    this.loadData()
  },
  methods: {
    async loadData() {
      try {
        const response = await this.$http.get('/web/users')
        if (response.success) {
          this.tableData = response.data
        }
      } catch (error) {
        this.$Message.error('加载数据失败')
      }
    }
  }
}
</script>

<style lang="less" scoped>
.component-name {
  padding: 20px;
}
</style>
```

## 开发注意事项

### 后端开发
- **执行环境要求** ⚠️ 极其重要：
  - 所有后端API相关命令必须在 `vige-api` 目录下执行
  - 执行前必须先启动虚拟环境：`pipenv shell`
  - 完整执行流程：`cd vige-api && pipenv shell && [命令]`
  - 禁止在项目根目录直接执行后端相关命令
- 新模块创建后在 `api/__init__.py` 中注册路由
- 数据模型在 `db.py` 中导入以便 Alembic 生成迁移
- 事务操作自动处理提交和回滚，无需手动 commit
- 所有需要认证的接口使用 `Depends(get_user)` 验证 JWT
- **严格按照接口类型使用正确的URL前缀**：Web端用`/web/`，后台管理用`/admin/`，微信端用`/wechat/`
- **必须使用对应的权限验证装饰器**：Web端用`@user_required`，后台管理用`@bo_required`，微信端用`@wechat_required`

### 前端开发
- 根据模块选择正确的 UI 框架 (iView vs Mint UI)
- API 调用处理错误情况
- 权限检查使用 `hasPermission` 方法
- 样式使用 Less 预处理器，添加 scoped 属性
- **不要调用不存在的接口，使用 TODO 标记需要实现的功能**
- **图片上传必须先获取 media_id，再绑定到业务对象**

### 跨端一致性
- API 接口使用端前缀区分
- 统一的响应格式和错误处理
- JWT + CSRF 双重认证保护
- 国际化支持 (后端 Flask-Babel, 前端 vue-i18n)

## 禁止事项
- ❌ **不要在新session中操作其他session的对象** ⚠️ 极其重要：绝对禁止在 `transaction_scope()` 中直接修改 `current_user` 等依赖注入对象，必须在新session中重新获取对象
- ❌ **不要混用不同session的数据库对象**：每个对象只能在其所属session中被操作，跨session操作会导致数据不一致
- ❌ **不要在 create() 方法中直接传入 ProfileMixin 属性** ⚠️ 极其重要：绝对禁止在创建对象时直接传入 `@string_property`、`@object_property` 等装饰器定义的字段，会导致 `'NoneType' object does not support item assignment` 错误，必须先创建对象再单独设置这些属性
- ❌ 不要混用数据库上下文 (Depends 和 transaction_scope)
- ❌ 不要在事务内手动调用 commit()
- ❌ 不要忽略用户认证 (缺少 Depends(get_user))
- ❌ 不要在前端直接使用错误的 UI 框架
- ❌ 不要调用不存在的接口，使用 TODO 标记
- ❌ 不要跳过 media 上传流程，直接使用图片URL
- ❌ **不要在 MediaModel 查询中添加用户关联条件**：严禁使用 `MediaModel.user_id` 等不存在的字段
- ❌ **不要在 MediaModel 中添加业务外键**：MediaModel 必须保持独立，不包含 user_id、product_id 等业务字段
- ❌ **不要错误理解 MediaModel 关联关系**：是其他表引用 MediaModel，不是 MediaModel 引用其他表
- ❌ 不要在接口参数中罗列大量参数，必须使用表单方式集中管理
- ❌ 不要在GET接口中使用过多单独的Query参数，优先使用Depends表单
- ❌ 不要直接抛出 HTTPException，必须使用标准化错误函数
- ❌ 不要使用自定义错误响应格式，必须遵循统一标准
- ❌ 不要混用HTTP状态码，确保状态码与错误类型匹配
- ❌ 不要使用错误的API前缀（Web端必须用/web/，后台管理必须用/admin/，微信端必须用/wechat/）
- ❌ 不要使用错误的权限验证装饰器（/web/用@user_required，/admin/用@bo_required，/wechat/用@wechat_required）
- ❌ 不要混用权限验证装饰器，必须与接口前缀严格对应
- ❌ **不要使用 Pydantic 1.x 语法**：严禁使用 `regex` 参数，必须使用 `pattern`
- ❌ **不要使用过时的配置语法**：严禁使用 `schema_extra`，必须使用 `json_schema_extra`
- ❌ **不要忽略 Pydantic 错误提示**：遇到 PydanticUserError 必须立即按规范修复

## 推荐做法
- ✅ 使用 Mixin 提供通用功能
- ✅ 事务操作使用上下文管理器
- ✅ **ProfileMixin 对象创建使用两步法**：先创建对象（只传入直接数据库字段），再单独设置 ProfileMixin 属性（`@string_property`、`@object_property` 等装饰器字段）
- ✅ API 响应格式保持一致
- ✅ 前端组件职责单一，结构清晰
- ✅ 添加必要的错误处理和日志记录
- ✅ 使用 TODO 标记需要实现的功能
- ✅ 图片上传先获取 media_id，再绑定业务对象
- ✅ 使用表单方式集中管理接口参数，避免参数列表过长
- ✅ GET接口优先使用 Depends(Form) 而不是多个 Query 参数
- ✅ 利用 Pydantic 表单提供完整的参数验证和 API 文档
- ✅ 使用标准化错误处理函数 (raise_user_not_found, raise_invalid_credentials 等)
- ✅ 在API路由中包含 STANDARD_ERROR_RESPONSES 配置
- ✅ 错误信息使用用户友好的中文描述
- ✅ 为每种错误类型提供明确的错误代码
- ✅ 严格按照接口类型使用正确的URL前缀（/web/、/admin/ 或 /wechat/）
- ✅ 根据接口前缀使用正确的权限验证装饰器（/web/→@user_required，/admin/→@bo_required，/wechat/→@wechat_required）
- ✅ 确保权限验证装饰器与接口类型严格对应，保证请求来源安全
- ✅ **使用 Pydantic 2.x 标准语法**：所有表单字段验证使用 `pattern` 而不是 `regex`
- ✅ **使用现代配置语法**：配置类中使用 `json_schema_extra` 而不是 `schema_extra`
- ✅ **定期检查兼容性**：创建表单类时参考标准模板，确保语法正确

## 文件命名规范
- 后端: snake_case (user_api.py, user_models.py)
- 前端: kebab-case 或 PascalCase (UserList.vue, user-detail.vue)
- 配置文件: 小写 (config.py, .env)

## 注释规范
- 中文注释，简洁明了
- 复杂业务逻辑必须添加注释
- API 接口添加功能说明
- 数据模型字段添加注释
- 使用 TODO 标记需要实现的功能

（说明）本规范面向 Vige 通用项目模板，示例涉及的具体业务仅作参考，可根据实际业务裁剪。

## 文档编写规范 ⭐ 重要
- **默认不写文档**：一般情况下，更新功能和代码时，不需要写文档
- **明确要求才写**：除非用户明确要求写文档，否则专注于代码实现
- **简洁实用**：如果需要写文档，保持简洁实用，重点突出关键信息
- **插件开发禁止文档**：mini_translate-extension 目录下禁止创建任何 .md 文档文件
- **专注代码实现**：插件功能更新时只修改代码，不创建说明文档 

## 网站设计规范 (vige-web) ⭐ 极其重要

### 设计理念与原则 🎨

#### 核心设计理念：
- **Apple Design 风格**：简洁、优雅、功能性至上
- **科技感与未来感**：体现AI翻译平台的专业性和先进性
- **用户体验优先**：直观易用，减少用户学习成本
- **一致性原则**：统一的视觉语言和交互模式

#### 设计原则：
1. **简洁至上**：去除不必要的装饰，突出核心功能
2. **层次清晰**：通过视觉层级引导用户注意力
3. **响应式设计**：适配各种设备和屏幕尺寸
4. **无障碍设计**：确保所有用户都能正常使用
5. **性能优化**：快速加载，流畅交互

### 色彩系统 🌈

#### 主色调：
```less
// 主品牌色 - 科技蓝
@primary-color: #007AFF;          // Apple 系统蓝
@primary-light: #5AC8FA;          // 浅蓝色
@primary-dark: #0051D5;           // 深蓝色

// 辅助色调
@secondary-color: #5856D6;        // 紫色，用于强调
@accent-color: #FF9500;           // 橙色，用于警示和亮点

// 中性色系 - Apple 风格灰度
@text-primary: #000000;           // 主要文字
@text-secondary: #3C3C43;         // 次要文字 (60% opacity)
@text-tertiary: #3C3C4399;       // 三级文字 (40% opacity)
@text-placeholder: #3C3C434D;     // 占位符文字 (30% opacity)

// 背景色系
@bg-primary: #FFFFFF;             // 主背景
@bg-secondary: #F2F2F7;           // 次要背景
@bg-tertiary: #FFFFFF;            // 卡片背景
@bg-elevated: #FFFFFF;            // 悬浮背景

// 分割线和边框
@separator-color: #3C3C4329;      // 分割线 (16% opacity)
@border-color: #3C3C432E;         // 边框 (18% opacity)

// 语义化颜色
@success-color: #34C759;          // 成功绿
@warning-color: #FF9500;          // 警告橙
@error-color: #FF3B30;            // 错误红
@info-color: #007AFF;             // 信息蓝
```

#### 深色模式支持：
```less
// 深色模式色彩（可选实现）
@dark-bg-primary: #000000;
@dark-bg-secondary: #1C1C1E;
@dark-bg-tertiary: #2C2C2E;
@dark-text-primary: #FFFFFF;
@dark-text-secondary: #EBEBF599;
```

### 字体系统 📝

#### 字体族：
```less
// Apple 系统字体栈
@font-family-base: -apple-system, BlinkMacSystemFont, 'Segoe UI', 'PingFang SC', 'Hiragino Sans GB', 'Microsoft YaHei', 'Helvetica Neue', Helvetica, Arial, sans-serif;

// 等宽字体（用于代码显示）
@font-family-mono: 'SF Mono', Monaco, 'Cascadia Code', 'Roboto Mono', Consolas, 'Courier New', monospace;
```

#### 字号层级：
```less
// 标题字号
@font-size-h1: 34px;              // 大标题
@font-size-h2: 28px;              // 二级标题  
@font-size-h3: 22px;              // 三级标题
@font-size-h4: 20px;              // 四级标题
@font-size-h5: 18px;              // 五级标题
@font-size-h6: 16px;              // 六级标题

// 正文字号
@font-size-large: 17px;           // 大号正文
@font-size-base: 15px;            // 基础正文
@font-size-small: 13px;           // 小号文字
@font-size-mini: 11px;            // 最小文字

// 字重
@font-weight-light: 300;          // 细体
@font-weight-normal: 400;         // 常规
@font-weight-medium: 500;         // 中等
@font-weight-semibold: 600;       // 半粗体
@font-weight-bold: 700;           // 粗体
```

#### 行高规范：
```less
@line-height-tight: 1.2;          // 紧凑行高（标题）
@line-height-base: 1.4;           // 基础行高（正文）
@line-height-relaxed: 1.6;        // 宽松行高（长文本）
```

### 间距系统 📏

#### 基础间距单位：
```less
// 4px 基础网格系统
@spacing-xs: 4px;                 // 超小间距
@spacing-sm: 8px;                 // 小间距
@spacing-md: 16px;                // 中等间距
@spacing-lg: 24px;                // 大间距
@spacing-xl: 32px;                // 超大间距
@spacing-xxl: 48px;               // 特大间距

// 组件内间距
@padding-xs: 8px;
@padding-sm: 12px;
@padding-md: 16px;
@padding-lg: 20px;
@padding-xl: 24px;

// 页面级间距
@container-padding: 20px;         // 容器内边距
@section-spacing: 40px;           // 区块间距
```

### 圆角和阴影系统 🔲

#### 圆角规范：
```less
@border-radius-sm: 6px;           // 小圆角
@border-radius-md: 8px;           // 中等圆角
@border-radius-lg: 12px;          // 大圆角
@border-radius-xl: 16px;          // 超大圆角
@border-radius-round: 50%;        // 圆形
```

#### 阴影系统：
```less
// Apple 风格阴影
@shadow-sm: 0 1px 3px rgba(0, 0, 0, 0.12), 0 1px 2px rgba(0, 0, 0, 0.24);
@shadow-md: 0 3px 6px rgba(0, 0, 0, 0.15), 0 2px 4px rgba(0, 0, 0, 0.12);
@shadow-lg: 0 10px 20px rgba(0, 0, 0, 0.15), 0 3px 6px rgba(0, 0, 0, 0.10);
@shadow-xl: 0 15px 25px rgba(0, 0, 0, 0.15), 0 5px 10px rgba(0, 0, 0, 0.05);

// 特殊阴影
@shadow-inset: inset 0 1px 2px rgba(0, 0, 0, 0.1);
@shadow-focus: 0 0 0 3px rgba(0, 122, 255, 0.3);
```

### 组件设计规范 🧩

#### 按钮样式：
```less
// 主要按钮
.btn-primary {
  background: @primary-color;
  color: #FFFFFF;
  border: none;
  border-radius: @border-radius-md;
  padding: 12px 20px;
  font-size: @font-size-base;
  font-weight: @font-weight-medium;
  transition: all 0.2s ease;
  
  &:hover {
    background: @primary-dark;
    transform: translateY(-1px);
    box-shadow: @shadow-md;
  }
  
  &:active {
    transform: translateY(0);
    box-shadow: @shadow-sm;
  }
}

// 次要按钮
.btn-secondary {
  background: @bg-secondary;
  color: @primary-color;
  border: 1px solid @border-color;
  border-radius: @border-radius-md;
  padding: 12px 20px;
  font-size: @font-size-base;
  font-weight: @font-weight-medium;
  transition: all 0.2s ease;
  
  &:hover {
    background: @primary-color;
    color: #FFFFFF;
    border-color: @primary-color;
  }
}

// 文字按钮
.btn-text {
  background: transparent;
  color: @primary-color;
  border: none;
  padding: 8px 12px;
  font-size: @font-size-base;
  font-weight: @font-weight-medium;
  transition: all 0.2s ease;
  
  &:hover {
    background: rgba(0, 122, 255, 0.1);
    border-radius: @border-radius-sm;
  }
}
```

#### 卡片组件：
```less
.card {
  background: @bg-tertiary;
  border-radius: @border-radius-lg;
  box-shadow: @shadow-sm;
  padding: @padding-lg;
  transition: all 0.3s ease;
  
  &:hover {
    box-shadow: @shadow-md;
    transform: translateY(-2px);
  }
  
  .card-header {
    margin-bottom: @spacing-md;
    padding-bottom: @spacing-md;
    border-bottom: 1px solid @separator-color;
  }
  
  .card-title {
    font-size: @font-size-h5;
    font-weight: @font-weight-semibold;
    color: @text-primary;
    margin: 0;
  }
  
  .card-content {
    color: @text-secondary;
    line-height: @line-height-base;
  }
}
```

#### 表单元素：
```less
.form-input {
  width: 100%;
  padding: 12px 16px;
  border: 1px solid @border-color;
  border-radius: @border-radius-md;
  font-size: @font-size-base;
  color: @text-primary;
  background: @bg-primary;
  transition: all 0.2s ease;
  
  &:focus {
    outline: none;
    border-color: @primary-color;
    box-shadow: @shadow-focus;
  }
  
  &::placeholder {
    color: @text-placeholder;
  }
}

.form-label {
  display: block;
  margin-bottom: @spacing-xs;
  font-size: @font-size-small;
  font-weight: @font-weight-medium;
  color: @text-secondary;
}
```

### 示例组件（可选） 🤖

#### 文档上传区域：
```less
.upload-area {
  border: 2px dashed @border-color;
  border-radius: @border-radius-lg;
  padding: @spacing-xxl;
  text-align: center;
  background: @bg-secondary;
  transition: all 0.3s ease;
  cursor: pointer;
  
  &:hover, &.drag-over {
    border-color: @primary-color;
    background: rgba(0, 122, 255, 0.05);
  }
  
  .upload-icon {
    font-size: 48px;
    color: @text-tertiary;
    margin-bottom: @spacing-md;
  }
  
  .upload-text {
    font-size: @font-size-large;
    color: @text-secondary;
    margin-bottom: @spacing-sm;
  }
  
  .upload-hint {
    font-size: @font-size-small;
    color: @text-tertiary;
  }
}
```

#### 进度指示器（示例）：
```less
.translation-progress {
  background: @bg-tertiary;
  border-radius: @border-radius-lg;
  padding: @padding-lg;
  box-shadow: @shadow-sm;
  
  .progress-header {
    display: flex;
    justify-content: space-between;
    align-items: center;
    margin-bottom: @spacing-md;
  }
  
  .progress-title {
    font-size: @font-size-base;
    font-weight: @font-weight-medium;
    color: @text-primary;
  }
  
  .progress-percentage {
    font-size: @font-size-small;
    color: @text-secondary;
  }
  
  .progress-bar {
    height: 6px;
    background: @bg-secondary;
    border-radius: 3px;
    overflow: hidden;
    
    .progress-fill {
      height: 100%;
      background: linear-gradient(90deg, @primary-color, @primary-light);
      border-radius: 3px;
      transition: width 0.3s ease;
    }
  }
}
```

#### 结果展示（示例）：
```less
.translation-result {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: @spacing-lg;
  margin-top: @spacing-lg;
  
  .result-panel {
    background: @bg-tertiary;
    border-radius: @border-radius-lg;
    padding: @padding-lg;
    box-shadow: @shadow-sm;
    
    .panel-header {
      display: flex;
      justify-content: space-between;
      align-items: center;
      margin-bottom: @spacing-md;
      padding-bottom: @spacing-sm;
      border-bottom: 1px solid @separator-color;
    }
    
    .panel-title {
      font-size: @font-size-base;
      font-weight: @font-weight-medium;
      color: @text-primary;
    }
    
    .panel-actions {
      display: flex;
      gap: @spacing-sm;
    }
    
    .panel-content {
      font-size: @font-size-base;
      line-height: @line-height-base;
      color: @text-secondary;
    }
  }
  
  // 响应式布局
  @media (max-width: 768px) {
    grid-template-columns: 1fr;
  }
}
```

### 动效和交互规范 ✨

#### 过渡动画：
```less
// 基础过渡
@transition-fast: 0.15s ease;
@transition-base: 0.2s ease;
@transition-slow: 0.3s ease;

// 缓动函数
@ease-out-quart: cubic-bezier(0.25, 1, 0.5, 1);
@ease-out-expo: cubic-bezier(0.19, 1, 0.22, 1);

// 通用过渡类
.transition-all {
  transition: all @transition-base;
}

.transition-transform {
  transition: transform @transition-base @ease-out-quart;
}

.transition-opacity {
  transition: opacity @transition-base;
}
```

#### 微交互效果：
```less
// 悬浮效果
.hover-lift {
  transition: transform @transition-base @ease-out-quart;
  
  &:hover {
    transform: translateY(-2px);
  }
}

// 点击反馈
.click-feedback {
  transition: transform @transition-fast;
  
  &:active {
    transform: scale(0.98);
  }
}

// 加载动画
@keyframes pulse {
  0%, 100% {
    opacity: 1;
  }
  50% {
    opacity: 0.5;
  }
}

.loading-pulse {
  animation: pulse 2s infinite;
}
```

### 响应式设计 📱

#### 断点系统：
```less
// 断点定义
@screen-xs: 480px;
@screen-sm: 768px;
@screen-md: 992px;
@screen-lg: 1200px;
@screen-xl: 1600px;

// 媒体查询 Mixin
.responsive(@size, @rules) {
  @media (max-width: @size) {
    @rules();
  }
}

// 使用示例
.container {
  max-width: 1200px;
  margin: 0 auto;
  padding: 0 @container-padding;
  
  .responsive(@screen-md, {
    padding: 0 @spacing-md;
  });
  
  .responsive(@screen-sm, {
    padding: 0 @spacing-sm;
  });
}
```

### CSS/Less 组织结构 📁

#### 文件组织：
```
styles/
├── variables/           # 变量定义
│   ├── colors.less     # 颜色变量
│   ├── typography.less # 字体变量
│   ├── spacing.less    # 间距变量
│   └── breakpoints.less # 断点变量
├── mixins/             # 混入函数
│   ├── buttons.less    # 按钮混入
│   ├── cards.less      # 卡片混入
│   └── responsive.less # 响应式混入
├── components/         # 组件样式
│   ├── buttons.less    # 按钮组件
│   ├── forms.less      # 表单组件
│   ├── cards.less      # 卡片组件
│   └── upload.less     # 上传组件
├── pages/             # 页面样式
│   ├── home.less      # 首页样式
│   ├── translate.less # 翻译页样式
│   └── profile.less   # 个人中心样式
├── layout/            # 布局样式
│   ├── header.less    # 头部样式
│   ├── sidebar.less   # 侧边栏样式
│   └── footer.less    # 底部样式
└── base/              # 基础样式
    ├── reset.less     # 样式重置
    ├── typography.less # 字体样式
    └── utilities.less  # 工具类
```

#### 命名规范：
```less
// BEM 命名方法
.block {}                    // 块
.block__element {}           // 元素
.block--modifier {}          // 修饰符
.block__element--modifier {} // 元素修饰符

// 示例
.translation-card {}                    // 翻译卡片
.translation-card__header {}            // 卡片头部
.translation-card__content {}           // 卡片内容
.translation-card--loading {}           // 加载状态
.translation-card__header--highlighted {} // 高亮头部
```

### 代码规范 📋

#### Less 编写规范：
```less
// ✅ 正确的样式编写方式
.translation-page {
  padding: @spacing-lg;
  background: @bg-primary;
  
  .page-header {
    margin-bottom: @spacing-xl;
    text-align: center;
    
    .page-title {
      font-size: @font-size-h2;
      font-weight: @font-weight-semibold;
      color: @text-primary;
      margin-bottom: @spacing-md;
    }
    
    .page-description {
      font-size: @font-size-base;
      color: @text-secondary;
      line-height: @line-height-base;
    }
  }
  
  .page-content {
    max-width: 800px;
    margin: 0 auto;
  }
  
  // 响应式设计
  @media (max-width: @screen-md) {
    padding: @spacing-md;
    
    .page-header .page-title {
      font-size: @font-size-h3;
    }
  }
}
```

#### 组件开发规范：
1. **单一职责**：每个组件只负责一个功能
2. **可复用性**：设计通用的组件接口
3. **一致性**：遵循统一的设计规范
4. **可访问性**：支持键盘导航和屏幕阅读器
5. **性能优化**：避免不必要的重绘和回流

#### 开发检查清单：
- ✅ 使用设计系统中定义的颜色变量
- ✅ 遵循字体和间距规范
- ✅ 实现响应式设计
- ✅ 添加适当的过渡动画
- ✅ 确保组件的可访问性
- ✅ 使用 BEM 命名规范
- ✅ 优化性能和加载速度
- ❌ 不要硬编码颜色值
- ❌ 不要使用内联样式
- ❌ 不要忽略移动端适配

### 特殊页面设计指南 📄

#### 首页设计：
- **英雄区域**：突出AI翻译的核心价值
- **功能展示**：清晰展示主要功能特性
- **上传入口**：显眼的文档上传区域
- **信任建立**：展示安全性和准确性

#### 翻译页面设计：
- **双栏布局**：原文和译文并列展示
- **工具栏**：提供复制、下载、分享等功能
- **进度指示**：清晰显示翻译进度
- **错误处理**：友好的错误提示和重试机制

#### 个人中心设计：
- **信息概览**：用户信息和使用统计
- **历史记录**：翻译历史的管理和查看
- **设置选项**：个性化设置和偏好配置

### Less语法规范 ⚠️ 重要

#### calc()函数使用规范：
在Less中使用`calc()`时必须使用转义语法，防止Less预处理器试图计算表达式：

```less
// ❌ 错误：Less会尝试计算表达式，导致编译错误
.container {
  height: calc(100vh - 80px);
  min-height: calc(100% - 40px);
}

// ✅ 正确：使用转义语法，告诉Less不要处理这个字符串
.container {
  height: ~"calc(100vh - 80px)";
  min-height: ~"calc(100% - 40px)";
}
```

#### 常见calc使用场景：
```less
// 视口高度计算
.main-content {
  min-height: ~"calc(100vh - 80px)";
}

// 容器宽度计算
.sidebar {
  width: ~"calc(100% - 280px)";
}

// 复杂表达式
.content-area {
  height: ~"calc(100vh - 140px)";
  padding: ~"calc(20px + 1rem)";
}
```

#### 重要注意事项：
1. **必须使用转义**: 所有包含计算的calc表达式都要用`~"calc(...)"`
2. **保持一致性**: 项目中统一使用这种语法
3. **编译检查**: 如果出现calc相关编译错误，检查是否缺少转义语法
4. **避免混用**: 不要在同一项目中混用转义和非转义的calc语法

### ES语法兼容性规范 ⚠️ 极其重要 

## 分步骤开发规范 ⚠️ 极其重要

### 核心原则：
分步骤、模块化的开发方式，确保每一步都经过确认再进行下一步，避免大规模返工。

### 开发流程规范：

#### 1. 模块开发步骤 ⭐ 必须遵循
每个新功能或模块的开发必须按以下顺序进行，**每完成一步都要等待用户确认**：

```
步骤1: 数据模型设计 (models.py + constants.py)
   ↓ 等待确认
步骤2: 表单验证设计 (forms.py)  
   ↓ 等待确认
步骤3: 响应模型设计 (schemas.py)
   ↓ 等待确认
步骤4: API接口实现 (api.py)
   ↓ 等待确认
步骤5: 后台任务实现 (tasks.py)
   ↓ 等待确认
步骤6: 业务逻辑集成和测试
```

#### 2. 单步骤完整实现原则 ⭐ 核心
- **最小功能单元**：每次只实现一个完整的最小功能板块
- **功能完整性**：确保每一步实现的功能在其范围内是完整可用的
- **逻辑清晰**：每一步的代码逻辑要完整、正确、符合预期
- **依赖明确**：每一步要明确其依赖关系和接口定义

#### 3. 代码提交规范：
```python
# ✅ 正确：单一职责，功能完整
# 第一步：只实现数据模型
class IMConversation(CRUDMixin, TrackableMixin):
    """IM会话表"""
    pass

# ✅ 正确：第二步才实现表单
class IMMessageForm(BaseModel):
    """IM消息表单"""
    pass

# ❌ 错误：一次性实现所有文件
# 不要同时提交 models.py + forms.py + api.py + tasks.py
```

#### 4. 确认节点规范：
每个步骤完成后，必须：
- **展示完整代码**：显示当前步骤的所有实现代码
- **说明实现逻辑**：解释为什么这样设计，解决了什么问题
- **列出下一步计划**：明确下一步要实现什么
- **等待明确确认**：用户明确表示"继续下一步"或"修改当前步骤"

#### 5. 错误处理和修正规范：
- **及时修正**：如果当前步骤不符合预期，立即修正，不要继续下一步
- **局部修改**：只修改当前步骤的代码，不要影响已确认的前序步骤
- **版本控制**：每个步骤都要保持代码的完整性和可追溯性

#### 6. 禁止行为 ❌：
- ❌ **不要一次性完成多个步骤**：避免大规模代码提交
- ❌ **不要跨步骤实现**：不要在实现models.py时同时写api.py的代码
- ❌ **不要假设确认**：没有得到明确确认不要进行下一步
- ❌ **不要调用测试工具**：避免运行命令、eslint修复、测试脚本等操作
- ❌ **不要写测试代码**：专注于业务逻辑实现，不写单元测试

#### 7. 推荐做法 ✅：
- ✅ **单一职责**：每次只关注一个文件或一个功能模块
- ✅ **渐进实现**：从数据层到表示层，自底向上实现
- ✅ **完整展示**：每步完成后展示完整的代码实现
- ✅ **逻辑说明**：解释设计思路和实现方案
- ✅ **依赖梳理**：明确当前步骤与其他模块的依赖关系

#### 8. 沟通模板：
```
## 第N步完成：[功能名称]

### 实现内容：
[完整代码展示]

### 设计说明：
[解释为什么这样设计]

### 下一步计划：
[明确下一步要实现的内容]

### 等待确认：
请确认当前实现是否符合预期，是否可以继续下一步？
```

#### 9. 特殊情况处理：
- **复杂模块拆分**：如果单个文件过于复杂，可以进一步细分步骤
- **依赖模块优先**：先实现被依赖的基础模块，再实现依赖模块
- **配置项分离**：配置相关的修改作为独立步骤处理

### 开发效率保障：
通过这种分步骤的方式，可以：
1. **减少返工**：每步确认避免大规模修改
2. **提高质量**：专注单一功能确保实现质量  
3. **便于调试**：问题定位更精准
4. **降低风险**：避免复杂逻辑错误传播
5. **提升沟通**：每步都有明确的交付和确认点

## FastAPI Path参数定义规范（按 Python 语法要求）

### 核心问题：
在FastAPI中定义Path参数时，**绝对不能为Path参数设置默认值语法**，这会导致 `non-default parameter follows default parameter` 语法错误。

### 错误做法（严禁）：
```python
# ❌ 错误：Path参数不能有默认值语法
async def get_task(
    task_id: int = Path(..., description="任务ID", example=1, gt=0),  # 错误！
    form: FilterForm = Depends(),  # 这个有默认值，放在Path参数后面会报错
    current_user: User = Depends(get_user)
):
    pass
```

### 正确做法：
```python
# ✅ 正确：Path参数使用简单类型注解
async def get_task(
    task_id: int,  # 简单方式：直接定义类型，FastAPI自动识别为Path参数
    form: FilterForm = Depends(),
    current_user: User = Depends(get_user)
):
    pass

# ✅ 正确：如果需要使用Path()验证，调整参数顺序
async def get_task(
    form: FilterForm = Depends(),
    current_user: User = Depends(get_user),
    task_id: int = Path(..., description="任务ID", example=1, gt=0)  # 放在最后
):
    pass
```

### 参数顺序规范：
1. **必需参数（无默认值）**：如简单Path参数、必需的表单参数
2. **可选参数（有默认值）**：如Depends()、Query()、Path()等

### 推荐的参数定义模式：
```python
# ✅ 最佳实践：简洁的Path参数定义
async def get_translation_task(
    task_id: int,  # 简单直接，FastAPI会自动识别为Path参数
    db: Session = Depends(sm.get_db),
    current_user: User = Depends(get_user)
):
    pass
```

### 重要注意事项：
1. **Path参数识别**：FastAPI会自动识别URL路径中的参数作为Path参数
2. **验证需求**：如果不需要特殊验证，直接使用简单的类型注解即可
3. **参数顺序**：有默认值的参数必须放在无默认值参数之后
4. **避免混淆**：Path(...)中的`...`不是默认值，但Python语法要求参数顺序

### 检查清单：
- ✅ Path参数使用简单类型注解（推荐）
- ✅ 如使用Path()，确保放在参数列表适当位置
- ✅ 有默认值的参数放在无默认值参数之后
- ❌ 不要为Path参数设置默认值语法
- ❌ 不要违反Python函数参数顺序规则

---
> Source: [Versus2017/vige](https://github.com/Versus2017/vige) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-06 -->
