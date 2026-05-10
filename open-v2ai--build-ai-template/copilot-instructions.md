## testing-rules

> 测试开发规范和最佳实践


# 测试开发规范

## 技术栈

- **后端**: pytest + pytest-asyncio + httpx
- **前端**: Jest + React Testing Library + Playwright
- **数据库**: 使用内存 SQLite 或独立的测试数据库
- **E2E**: Playwright

## 测试文件组织

测试文件应与源代码文件并置或存放在专门的 `tests` 目录中。

### 后端测试结构

```bash
api/tests/
├── conftest.py              # Pytest 配置和 fixtures
├── test_main.py             # 主应用测试
├── test_auth.py             # 认证流程测试
└── crud/
    └── test_user_crud.py
```

### 前端测试结构

```bash
web/
├── __tests__/              # 组件单元测试
│   ├── components/
│   │   └── chat-content-page.test.tsx
│   └── utils/
│       └── fetcher.test.ts
└── e2e/                    # 端到端测试
    └── auth.spec.ts
```

## 后端测试规范 (pytest)

### 核心 Fixtures (`conftest.py`)

使用 Fixtures 来设置测试环境，如数据库连接和 API 测试客户端。

```python
import pytest
from fastapi.testclient import TestClient
from sqlmodel import Session, SQLModel, create_engine
from sqlmodel.pool import StaticPool

from app.main import app
from app.dependencies.db import get_db
from app.models.user import User
from app.core.security import create_access_token

@pytest.fixture(name="session")
def session_fixture():
    """创建一个使用内存 SQLite 的独立数据库会话。"""
    engine = create_engine(
        "sqlite://",
        connect_args={"check_same_thread": False},
        poolclass=StaticPool
    )
    SQLModel.metadata.create_all(engine)
    with Session(engine) as session:
        yield session

@pytest.fixture(name="client")
def client_fixture(session: Session):
    """创建一个 FastAPI 测试客户端，并覆盖数据库依赖。"""
    app.dependency_overrides[get_db] = lambda: session
    with TestClient(app) as client:
        yield client
    app.dependency_overrides.clear()

@pytest.fixture
def test_user(session: Session) -> User:
    """创建一个普通测试用户并存入数据库。"""
    # ... 创建用户的代码 ...
    return user

@pytest.fixture
def auth_headers(test_user: User) -> dict:
    """为测试用户生成认证头。"""
    token = create_access_token(subject=test_user.id)
    return {"Authorization": f"Bearer {token}"}
```

### API 测试示例

专注于测试 API 的请求和响应是否符合预期。

```python
# api/tests/test_auth.py
from fastapi.testclient import TestClient

def test_login_success(client: TestClient, test_user: User):
    """测试用户使用正确凭据成功登录。"""
    response = client.post("/api/v1/auth/login", json={
        "email": test_user.email,
        "password": "testpassword123"
    })

    assert response.status_code == 200
    data = response.json()
    assert "access_token" in data
    assert data["user"]["id"] == test_user.id

def test_get_current_user_unauthorized(client: TestClient):
    """测试未提供认证头时访问受保护路由。"""
    response = client.get("/api/v1/users/me")
    assert response.status_code == 401
```

## 前端测试规范

### 组件测试 (React Testing Library)

测试组件的渲染和交互行为，避免测试实现细节。

```typescript
// web/__tests__/components/chat-input.test.tsx
import { render, screen, fireEvent } from '@testing-library/react'
import ChatInput from '@/components/chat-input'

describe('ChatInput', () => {
  it('应该在点击按钮时调用 onSend 回调', () => {
    const handleSend = jest.fn()
    render(<ChatInput onSend={handleSend} />)

    const input = screen.getByRole('textbox')
    const sendButton = screen.getByRole('button', { name: /send/i })

    // 模拟用户输入和点击
    fireEvent.change(input, { target: { value: '你好' } })
    fireEvent.click(sendButton)

    // 断言回调被正确调用
    expect(handleSend).toHaveBeenCalledWith('你好')
    // 断言输入框在发送后被清空
    expect(input.value).toBe('')
  })
})
```

### E2E 测试 (Playwright)

模拟真实用户操作，测试完整的业务流程。

```typescript
// web/e2e/auth.spec.ts
import { test, expect } from '@playwright/test'

test('用户登录和退出流程', async ({ page }) => {
  // 访问登录页面
  await page.goto('/login')

  // 填写表单并登录
  await page.fill('input[name="email"]', 'test@example.com')
  await page.fill('input[name="password"]', 'password123')
  await page.click('button[type="submit"]')

  // 验证登录成功
  await expect(page).toHaveURL('/')
  await expect(page.locator('[data-testid="user-avatar"]')).toBeVisible()

  // 退出登录
  await page.click('[data-testid="user-avatar"]')
  await page.click('text=退出登录')

  // 验证退出成功
  await expect(page).toHaveURL('/login')
})
```

## 测试命令

### 后端

```bash
# 进入后端目录
cd api/

# 运行所有测试
uv run pytest

# 运行特定文件
uv run pytest tests/test_auth.py

# 生成覆盖率报告
uv run pytest --cov=app
```

### 前端

```bash
# 进入前端目录
cd web/

# 运行单元测试
pnpm test

# 运行 E2E 测试
pnpm test:e2e
```

## CI/CD 集成

在 GitHub Actions 中自动化测试流程，确保代码质量。

```yaml
# .github/workflows/test.yml
name: 测试

on: [push, pull_request]

jobs:
  backend-tests:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with:
          python-version: '3.12'
      - name: 安装依赖并运行测试
        run: |
          cd api
          pip install uv
          uv sync
          uv run pytest

  frontend-tests:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: pnpm/action-setup@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'pnpm'
      - name: 安装依赖并运行测试
        run: |
          cd web
          pnpm install
          pnpm test -- --watchAll=false
          pnpm build
          pnpm test:e2e
```

## 最佳实践

- **AAA 模式**: 测试应遵循 Arrange（准备）、Act（执行）、Assert（断言）的结构。
- **独立性与可重复性**: 每个测试用例都应能独立运行，且结果稳定一致。
- **覆盖关键路径**: 优先为核心业务逻辑和复杂场景编写测试。
- **Mock 外部依赖**: 单元测试中应 Mock 掉数据库、外部 API 等依赖，以保证测试速度和稳定性。
- **测试命名**: 测试名称应清晰描述被测试的功能、场景和预期结果，如 `test_login_with_invalid_password_returns_401`。
- **测试覆盖率**: 目标代码覆盖率应不低于 70%，并通过 CI/CD 强制执行。
- **错误场景测试**: 除了正常流程，还必须覆盖各种预期的错误和边界情况。

---
> Source: [open-v2ai/build-ai-template](https://github.com/open-v2ai/build-ai-template) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-06 -->
