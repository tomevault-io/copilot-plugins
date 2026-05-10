## quick-comments-guide

> 代码注释快速指南 - 简明扼要的注释规范参考


# 代码注释快速指南

## 核心要求

- **必须使用中文注释**
- **每个函数、类、组件和模型都应有详细的文档字符串 (docstring/JSDoc)**
- **复杂或关键的业务逻辑必须有行内注释**
- **代码修改时必须同步更新注释**

## Python 注释模板

### 函数/方法

```python
async def create_user(db: Session, user_data: UserCreate) -> User:
    """
    创建新用户并处理相关业务逻辑。

    - 参数:
      - `db`: 数据库会话对象。
      - `user_data`: 用户创建数据 (email, password)。
    - 返回:
      - 创建成功的用户对象。
    - 异常:
      - `ValueError`: 邮箱已存在。
    """
```

### 类

```python
class UserService:
    """
    处理用户相关业务逻辑的服务类。

    主要功能:
    - 用户注册与登录
    - 权限验证
    - 邮箱验证码处理
    """
```

## TypeScript/React 注释模板

### 组件/Hook

```typescript
/**
 * 聊天内容页面组件，负责显示消息历史和处理用户输入。
 *
 * @param chatId - 聊天会话 ID。
 * @param agentId - AI 助手 ID。
 */
export default function ChatContentPage({ chatId, agentId }: ChatContentPageProps) {
  // ...
}

/**
 * 管理聊天消息状态的 Hook，包括加载、发送和本地缓存。
 *
 * @param chatId - 聊天会话 ID。
 * @returns 消息管理相关的状态和函数。
 */
export function useChatMessages(chatId: number) {
  // ...
}
```

### 状态变量

```typescript
// 消息列表状态，存储当前对话的所有消息
const [messages, setMessages] = useState<Message[]>([])

// 发送状态，用于控制按钮禁用和加载指示器
const [isSending, setIsSending] = useState(false)
```

## API 路由和数据库模型

### API 路由 (FastAPI)

```python
@router.post("/users", response_model=UserResponse)
async def create_user_endpoint(user_data: UserCreate):
    """
    创建用户 API 端点。

    - 权限: 管理员
    - 请求: `user_data` (包含 email, password)
    - 响应 (201): 创建成功的用户信息。
    - 响应 (400): 邮箱重复或数据格式错误。
    """
```

### 数据库模型 (SQLModel)

```python
class User(SQLModel, table=True):
    """
    用户数据模型。

    业务规则:
    - 邮箱是唯一标识符，用于登录。
    - 支持软删除 (`is_deleted` 字段)。
    - 密码必须加密存储。
    """
    id: Optional[int] = Field(primary_key=True, description="用户唯一标识符")
    email: str = Field(unique=True, description="用户邮箱地址，登录凭证")
```

## 注释清单

- [ ] **该注释吗？**: 所有公共函数、组件、模型、API 端点和复杂逻辑都需要注释。
- [ ] **写了什么？**: 注释应清晰说明代码的**功能**、**目的**和**业务规则**。
- [ ] **质量如何？**:
  - 使用中文。
  - 描述参数和返回值。
  - 列出可能的异常或错误情况。
- [ ] **是否同步？**: 修改代码后，务必检查并更新相关注释。

---
> Source: [open-v2ai/build-ai-template](https://github.com/open-v2ai/build-ai-template) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-06 -->
