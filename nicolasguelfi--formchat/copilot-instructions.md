## formchat

> Use SQLAlchemy ORM for database operations:

# Database and Caching Rules

## Database Design
Use SQLAlchemy ORM for database operations:

```python
from sqlalchemy import Column, Integer, String, DateTime, Text, Boolean
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import sessionmaker
from datetime import datetime

Base = declarative_base()

class FormSession(Base):
    __tablename__ = "form_sessions"
    
    id = Column(String, primary_key=True)
    form_id = Column(String, nullable=False)
    user_id = Column(String, nullable=True)
    data = Column(Text, nullable=False)  # JSON data
    status = Column(String, default="in_progress")
    created_at = Column(DateTime, default=datetime.utcnow)
    updated_at = Column(DateTime, default=datetime.utcnow, onupdate=datetime.utcnow)
    completed_at = Column(DateTime, nullable=True)
```

## Connection Management
```python
from sqlalchemy import create_engine
from sqlalchemy.pool import QueuePool
from contextlib import asynccontextmanager

# Database engine with connection pooling
engine = create_engine(
    DATABASE_URL,
    poolclass=QueuePool,
    pool_size=20,
    max_overflow=30,
    pool_pre_ping=True,
    pool_recycle=3600
)

@asynccontextmanager
async def get_db_session():
    """Get database session with proper cleanup"""
    session = SessionLocal()
    try:
        yield session
        session.commit()
    except Exception:
        session.rollback()
        raise
    finally:
        session.close()
```

## Redis Caching
```python
import redis
import json
from typing import Optional, Dict, Any

class CacheManager:
    def __init__(self, redis_client: redis.Redis):
        self.redis = redis_client
    
    async def get_session(self, session_id: str) -> Optional[Dict[str, Any]]:
        """Get session data from cache"""
        key = f"session:{session_id}"
        data = self.redis.get(key)
        return json.loads(data) if data else None
    
    async def set_session(self, session_id: str, data: Dict[str, Any], ttl: int = 3600):
        """Set session data in cache"""
        key = f"session:{session_id}"
        self.redis.setex(key, ttl, json.dumps(data))
    
    async def get_form_schema(self, form_id: str) -> Optional[Dict[str, Any]]:
        """Get form schema from cache"""
        key = f"schema:{form_id}"
        data = self.redis.get(key)
        return json.loads(data) if data else None
    
    async def set_form_schema(self, form_id: str, schema: Dict[str, Any], ttl: int = 86400):
        """Cache form schema"""
        key = f"schema:{form_id}"
        self.redis.setex(key, ttl, json.dumps(schema))
```

## Data Encryption
```python
from cryptography.fernet import Fernet
import base64

class DataEncryption:
    def __init__(self, key: bytes):
        self.cipher = Fernet(key)
    
    def encrypt_data(self, data: str) -> str:
        """Encrypt sensitive data"""
        encrypted = self.cipher.encrypt(data.encode())
        return base64.b64encode(encrypted).decode()
    
    def decrypt_data(self, encrypted_data: str) -> str:
        """Decrypt sensitive data"""
        decoded = base64.b64decode(encrypted_data.encode())
        decrypted = self.cipher.decrypt(decoded)
        return decrypted.decode()
```

## Database Migrations
```python
from alembic import command
from alembic.config import Config

def run_migrations():
    """Run database migrations"""
    alembic_cfg = Config("alembic.ini")
    command.upgrade(alembic_cfg, "head")

def create_migration(message: str):
    """Create new migration"""
    alembic_cfg = Config("alembic.ini")
    command.revision(alembic_cfg, message=message, autogenerate=True)
```

## Query Optimization
```python
from sqlalchemy.orm import selectinload, joinedload

# Eager loading for related data
def get_form_session_with_data(session_id: str):
    return db.query(FormSession)\
        .options(selectinload(FormSession.form_data))\
        .filter(FormSession.id == session_id)\
        .first()

# Use indexes for frequently queried fields
class FormSession(Base):
    __tablename__ = "form_sessions"
    
    id = Column(String, primary_key=True, index=True)
    form_id = Column(String, nullable=False, index=True)
    user_id = Column(String, nullable=True, index=True)
    status = Column(String, default="in_progress", index=True)
```

## Connection Pooling
```python
from sqlalchemy.pool import QueuePool

# Configure connection pool
engine = create_engine(
    DATABASE_URL,
    poolclass=QueuePool,
    pool_size=20,          # Number of connections to maintain
    max_overflow=30,       # Additional connections when needed
    pool_pre_ping=True,    # Verify connections before use
    pool_recycle=3600,     # Recycle connections after 1 hour
    echo=False             # Set to True for SQL logging
)
```

## Redis Configuration
```python
import redis
from redis.sentinel import Sentinel

# Single Redis instance
redis_client = redis.Redis(
    host='localhost',
    port=6379,
    db=0,
    decode_responses=True,
    socket_connect_timeout=5,
    socket_timeout=5,
    retry_on_timeout=True
)

# Redis Sentinel for high availability
sentinel = Sentinel([
    ('redis-sentinel-1', 26379),
    ('redis-sentinel-2', 26379),
    ('redis-sentinel-3', 26379)
])

redis_client = sentinel.master_for(
    'mymaster',
    socket_timeout=0.1,
    decode_responses=True
)
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/nicolasguelfi) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
