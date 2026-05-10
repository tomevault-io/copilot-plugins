## security-rules

> 安全开发规范和最佳实践


# 安全开发规范

## 核心安全原则

- **最小权限原则**: 用户和服务只获得完成任务所需的最小权限。
- **深度防御**: 实施多层安全控制，不依赖单一安全措施。
- **默认安全**: 默认配置应为最安全的配置。
- **失败安全**: 系统故障时应拒绝访问，而非允许访问。
- **数据保护**: 敏感数据必须在存储和传输过程中加密。

## 认证和授权

### JWT 与密码安全

使用 JWT 进行无状态认证，并对用户密码进行强哈希处理。

位置: `api/app/core/security.py`

```python
from datetime import datetime, timedelta
from jose import jwt
from passlib.context import CryptContext
import secrets

# 使用 bcrypt 算法进行密码哈希
pwd_context = CryptContext(schemes=["bcrypt"], deprecated="auto")

def create_access_token(subject: str, expires_delta: timedelta) -> str:
    """创建 JWT 访问令牌，包含主题和过期时间。"""
    expire = datetime.utcnow() + expires_delta
    to_encode = {
        "sub": subject,
        "exp": expire,
        "jti": secrets.token_hex(16) # 防止重放攻击
    }
    return jwt.encode(to_encode, "YOUR_SECRET_KEY", algorithm="HS256")

def verify_password(plain_password: str, hashed_password: str) -> bool:
    """验证明文密码和哈希密码是否匹配。"""
    return pwd_context.verify(plain_password, hashed_password)

def hash_password(password: str) -> str:
    """对密码进行哈希处理。"""
    return pwd_context.hash(password)
```

### 权限控制

通过依赖注入实现基于角色的权限控制。

位置: `api/app/dependencies/auth.py`

```python
from fastapi import Depends, HTTPException, status
from fastapi.security import HTTPBearer
from ..models.user import User, UserType
from ..crud.user import user_crud

security = HTTPBearer()

def get_current_user(
    credentials = Depends(security),
    db = Depends(get_db)
) -> User:
    """依赖项：获取当前认证用户。"""
    token_payload = verify_token(credentials.credentials)
    if not token_payload or not (user_id := token_payload.get("sub")):
        raise HTTPException(status_code=status.HTTP_401_UNAUTHORIZED, detail="无效的 Token")

    user = user_crud.get(db, id=int(user_id))
    if not user or not user.is_active or user.is_deleted:
        raise HTTPException(status_code=status.HTTP_401_UNAUTHORIZED, detail="用户不存在或未激活")

    return user

def get_current_admin_user(current_user: User = Depends(get_current_user)) -> User:
    """依赖项：确保当前用户是管理员。"""
    if current_user.user_type != UserType.ADMIN:
        raise HTTPException(status_code=status.HTTP_403_FORBIDDEN, detail="需要管理员权限")
    return current_user
```

## 输入验证和防护

### SQL 注入防护

所有数据库操作必须使用参数化查询，禁止拼接 SQL 字符串。

```python
# api/app/crud/base.py

# 正确示例：使用 ORM 和参数化查询
def get_user_by_email(db: Session, email: str) -> Optional[User]:
    statement = select(User).where(User.email == email)
    return db.exec(statement).first()

# 错误示例：禁止使用字符串拼接
def unsafe_get_user(db: Session, email: str):
    # 极易受到 SQL 注入攻击！
    query = f"SELECT * FROM user WHERE email = '{email}'"
    ...
```

### XSS 防护

对所有用户生成的内容（UGC）进行清理，对输出到 HTML 的数据进行转义。

```python
# api/app/utils/sanitizer.py
import bleach
import html

# 允许的 HTML 标签和属性白名单
ALLOWED_TAGS = ['p', 'br', 'strong', 'em', 'ol', 'ul', 'li', 'a']
ALLOWED_ATTRIBUTES = {'a': ['href', 'title']}

def sanitize_html(content: str) -> str:
    """清理用户输入的 HTML，移除潜在的 XSS 攻击代码。"""
    return bleach.clean(
        content,
        tags=ALLOWED_TAGS,
        attributes=ALLOWED_ATTRIBUTES,
        strip=True
    )

def escape_html(text: str) -> str:
    """对输出到模板的字符串进行 HTML 转义。"""
    return html.escape(text)
```

## HTTPS 和传输安全

生产环境必须强制使用 HTTPS。在 Nginx 中配置 SSL/TLS，并添加安全头。

```nginx
# deploy/nginx/default.conf

# 强制跳转到 HTTPS
server {
    listen 80;
    server_name your-domain.com;
    return 301 https://$host$request_uri;
}

server {
    listen 443 ssl http2;
    server_name your-domain.com;

    # SSL 配置
    ssl_certificate /etc/letsencrypt/live/your-domain.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/your-domain.com/privkey.pem;
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_prefer_server_ciphers off;

    # 安全头
    add_header Strict-Transport-Security "max-age=31536000; includeSubDomains; preload" always;
    add_header X-Frame-Options "SAMEORIGIN" always;
    add_header X-Content-Type-Options "nosniff" always;
    add_header Referrer-Policy "strict-origin-when-cross-origin" always;
}
```

## 数据保护

### 敏感数据加密与脱敏

- **加密**: 对存储在数据库中的敏感信息（如 API密钥、第三方服务凭证）进行对称加密。
- **脱敏**: 在日志和对外展示中，对敏感信息（如邮箱、手机号）进行脱敏处理。

```python
# 示例：数据脱敏
def mask_email(email: str) -> str:
    """邮箱脱敏处理，例如: u***@example.com"""
    if '@' not in email:
        return email
    local, domain = email.split('@', 1)
    return f"{local[0]}{'*' * (len(local) - 1)}@{domain}"
```

## 安全监控与审计

### 安全日志和入侵检测

- **安全日志**: 记录关键安全事件，如登录成功/失败、权限变更、重要数据访问等。
- **入侵检测**: 实现简单的暴力破解防御机制，限制来自同一 IP 的登录失败次数。

```python
# 伪代码：暴力破解检测
from collections import defaultdict

login_attempts = defaultdict(list)
MAX_ATTEMPTS = 5
LOCKOUT_PERIOD = 900 # 15分钟

def record_failed_login(ip: str):
    """记录失败登录并检查是否需要封禁。"""
    now = time.time()
    attempts = login_attempts[ip]

    # 移除过期的尝试记录
    attempts = [t for t in attempts if now - t < LOCKOUT_PERIOD]

    attempts.append(now)
    login_attempts[ip] = attempts

    if len(attempts) >= MAX_ATTEMPTS:
        ban_ip(ip, LOCKOUT_PERIOD)
        return True
    return False
```

## 安全最佳实践清单

- **认证授权**: 使用强密码策略和基于角色的访问控制（RBAC），JWT Token 必须设置合理的过期时间。
- **输入验证**: 对所有用户输入进行验证和清理，使用参数化查询防止 SQL 注入，实施 XSS 防护。
- **数据保护**: 敏感数据加密存储，强制使用 HTTPS，日志和API响应中对敏感数据进行脱敏。
- **依赖管理**: 定期扫描并更新项目依赖（`uv sync`, `pnpm update`），避免使用存在已知漏洞的库。
- **错误处理**: 避免在生产环境中泄露详细的错误信息和堆栈跟踪。
- **基础设施**: 及时更新服务器和容器镜像，配置严格的防火墙规则，最小化暴露端口。
- **监控审计**: 记录所有关键安全事件，监控异常访问模式，定期进行安全审计。
- **密钥管理**: `SECRET_KEY` 和其他敏感凭证必须通过环境变量配置，禁止硬编码在代码中。

---
> Source: [open-v2ai/build-ai-template](https://github.com/open-v2ai/build-ai-template) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-06 -->
