## ai-agent-development

> AI 和 Agent 开发规范和 OpenAI API 集成最佳实践


# AI 和 Agent 开发规范

## 核心架构

```bash
api/app/
├── agents/          # Agent 实现
│   ├── llm.py      # LLM 基础封装
│   └── dify.py     # Dify 集成
└── utils/           # AI 工具模块
```

## LLM 集成规范

### OpenAI API 配置

位置：[api/app/agents/llm.py](mdc:api/app/agents/llm.py)

```python
from openai import AsyncOpenAI
from typing import AsyncGenerator, Dict, Any, List, Optional
import logging
import asyncio
from datetime import datetime

from ..core.config import settings
from ..core.exceptions import LLMException

logger = logging.getLogger(__name__)

class LLMService:
    """
    LLM 服务类

    功能:
    - OpenAI API 封装
    - 流式响应处理
    - 错误重试机制
    - Token 使用量统计
    """

    def __init__(self):
        """
        初始化 LLM 服务

        配置:
        - API Key 从环境变量获取
        - 设置超时和重试参数
        - 初始化客户端
        """
        self.client = AsyncOpenAI(
            api_key=settings.AGENT_API_KEY,
            base_url=settings.AGENT_BASE_URL,
            timeout=30.0,
            max_retries=3
        )

        # 默认模型配置
        self.default_model = settings.AGENT_MODEL_NAME or "gpt-4o-mini"
        self.default_temperature = 0.7
        self.default_max_tokens = 2000

    async def create_chat_completion(
        self,
        messages: List[Dict[str, str]],
        model: Optional[str] = None,
        stream: bool = False,
        temperature: Optional[float] = None,
        max_tokens: Optional[int] = None,
        **kwargs
    ):
        """
        创建聊天完成请求

        参数:
        - messages: 消息列表
        - model: 模型名称
        - stream: 是否流式输出
        - temperature: 温度参数
        - max_tokens: 最大 token 数

        返回:
        - OpenAI 响应对象

        异常:
        - LLMException: API 调用失败
        """
        try:
            # 使用默认值填充参数
            model = model or self.default_model
            temperature = temperature if temperature is not None else self.default_temperature
            max_tokens = max_tokens or self.default_max_tokens

            logger.info(f"调用 LLM API: model={model}, stream={stream}, messages_count={len(messages)}")

            response = await self.client.chat.completions.create(
                model=model,
                messages=messages,
                stream=stream,
                temperature=temperature,
                max_tokens=max_tokens,
                **kwargs
            )

            return response

        except Exception as e:
            logger.error(f"OpenAI API 调用失败: {e}")
            raise LLMException(f"AI 服务调用失败: {str(e)}")

# 全局 LLM 服务实例
llm_service = LLMService()
```

### 流式响应处理

位置：[api/app/routers/v1/chat.py](mdc:api/app/routers/v1/chat.py)

```python
from fastapi.responses import StreamingResponse
import json
import asyncio
from typing import AsyncGenerator
from sqlalchemy.orm import Session

from ...agents.llm import llm_service
from ...crud.message import create_message, get_all_messages
from ...crud.chat import get_chat_by_id
from ...models.message import Message

async def stream_chat_response(
    messages: List[Dict[str, str]],
    agent_config: Dict[str, Any],
    chat_id: int,
    user_id: int,
    db: Session
) -> AsyncGenerator[str, None]:
    """
    流式聊天响应生成器

    功能:
    - 实时流式输出 AI 响应
    - 统计 Token 使用量
    - 保存消息到数据库
    - 错误处理和重试

    参数:
    - messages: 对话消息列表
    - agent_config: Agent 配置
    - chat_id: 对话 ID
    - user_id: 用户 ID
    - db: 数据库会话

    生成:
    - SSE 格式的流式数据
    """
    full_response = ""
    token_usage = {
        "prompt_tokens": 0,
        "completion_tokens": 0,
        "total_tokens": 0
    }

    try:
        # 构建完整的消息上下文
        system_prompt = agent_config.get("system_prompt", "")
        if system_prompt:
            full_messages = [{"role": "system", "content": system_prompt}] + messages
        else:
            full_messages = messages

        # 获取模型配置
        model_conf = agent_config.get("model_conf", {})
        model = model_conf.get("model", "gpt-4o-mini")
        temperature = model_conf.get("temperature", 0.7)
        max_tokens = model_conf.get("max_tokens", 2000)

        logger.info(f"开始流式响应生成: chat_id={chat_id}, model={model}")

        # 调用 OpenAI 流式 API
        stream = await llm_service.create_chat_completion(
            messages=full_messages,
            model=model,
            stream=True,
            temperature=temperature,
            max_tokens=max_tokens
        )

        # 流式输出处理
        async for chunk in stream:
            if hasattr(chunk, 'choices') and chunk.choices:
                choice = chunk.choices[0]

                # 处理内容增量
                if hasattr(choice, 'delta') and choice.delta.content:
                    content = choice.delta.content
                    full_response += content

                    # 发送内容块
                    yield f"data: {json.dumps({
                        'content': content,
                        'type': 'content',
                        'chat_id': chat_id,
                        'timestamp': datetime.utcnow().isoformat()
                    }, ensure_ascii=False)}\n\n"

                    # 控制输出速度，提升用户体验
                    await asyncio.sleep(0.02)

                # 处理完成信息和 Token 统计
                if hasattr(choice, 'finish_reason') and choice.finish_reason:
                    if hasattr(chunk, 'usage') and chunk.usage:
                        token_usage = {
                            "prompt_tokens": chunk.usage.prompt_tokens,
                            "completion_tokens": chunk.usage.completion_tokens,
                            "total_tokens": chunk.usage.total_tokens
                        }

        # 保存助手消息到数据库
        if full_response:
            assistant_message = await create_message(
                db=db,
                chat_id=chat_id,
                user_id=user_id,
                role="assistant",
                content=full_response,
                token_usage=token_usage.get("total_tokens", 0)
            )

            logger.info(f"助手消息已保存: message_id={assistant_message.id}, tokens={token_usage.get('total_tokens', 0)}")

        # 发送完成标记
        yield f"data: {json.dumps({
            'type': 'done',
            'chat_id': chat_id,
            'message': full_response,
            'token_usage': token_usage,
            'timestamp': datetime.utcnow().isoformat()
        }, ensure_ascii=False)}\n\n"

    except Exception as e:
        logger.error(f"流式响应生成失败: chat_id={chat_id}, error={e}", exc_info=True)

        # 发送错误信息
        error_msg = "AI 响应生成失败，请稍后重试"
        yield f"data: {json.dumps({
            'error': error_msg,
            'type': 'error',
            'chat_id': chat_id,
            'timestamp': datetime.utcnow().isoformat()
        }, ensure_ascii=False)}\n\n"

# Server-Sent Events 端点
@router.post("/stream")
async def stream_chat_endpoint(
    request: ChatStreamRequest,
    db: Session = Depends(get_db),
    current_user = Depends(get_current_user)
):
    # 获取或创建对话
    chat = await get_or_create_chat(db, request.chat_id, current_user.id)

    # 获取 Agent 配置
    agent = await get_agent(db, request.agent_id or 1)
    if not agent or not agent.is_active:
        raise HTTPException(status_code=404, detail="Agent not found")

    # 保存用户消息
    await save_message(db, chat.id, "user", request.message)

    # 获取对话历史
    messages = await get_chat_messages(db, chat.id, limit=20)
    openai_messages = [
        {"role": msg.role, "content": msg.content}
        for msg in messages
    ]

    # 构建 Agent 配置
    agent_config = {
        "system_prompt": agent.system_prompt,
        "model_conf": agent.model_conf or {}
    }

    return StreamingResponse(
        stream_chat_response(openai_messages, agent_config, chat.id, db),
        media_type="text/plain",
        headers={
            "Cache-Control": "no-cache",
            "Connection": "keep-alive",
            "Content-Type": "text/event-stream"
        }
    )
```

## Agent 配置管理

### Agent 模型定义

```python
from sqlmodel import SQLModel, Field
from typing import Optional, Dict, Any
from datetime import datetime

class Agent(SQLModel, table=True):
    id: Optional[int] = Field(default=None, primary_key=True)
    name: str = Field(max_length=100, description="Agent 名称")
    description: Optional[str] = Field(default=None, description="Agent 描述")
    system_prompt: str = Field(description="系统提示词")
    model_conf: Optional[Dict[str, Any]] = Field(
        default=None,
        description="模型配置（JSON格式）"
    )
    is_active: bool = Field(default=True, description="是否激活")
    created_at: datetime = Field(default_factory=datetime.utcnow)
    updated_at: datetime = Field(default_factory=datetime.utcnow)
    is_deleted: bool = Field(default=False, description="软删除标记")
```

### Agent 预设配置

位置：`api/app/agents/presets.py` (示例)

```python
AGENT_PRESETS = {
    "general": {
        "name": "通用助手",
        "description": "适用于日常对话和问答的通用 AI 助手",
        "system_prompt": "你是一个友善、有帮助的 AI 助手...",
        "model_conf": { "model": "gpt-4o-mini", "temperature": 0.7, ... },
    },
    "coding": {
        "name": "编程助手",
        "description": "专业的编程和技术开发助手",
        "system_prompt": "你是一个专业的编程助手，具备以下能力...",
        "model_conf": { "model": "gpt-4o-mini", "temperature": 0.2, ... },
    },
    # ... 其他预设
}
```

### 配置验证

```python
from pydantic import BaseModel, validator

class AgentModelConfig(BaseModel):
    model: str = "gpt-4o-mini"
    temperature: float = 0.7
    max_tokens: int = 2000
    top_p: float = 1.0

    @validator('temperature')
    def validate_temperature(cls, v):
        if not 0 <= v <= 2:
            raise ValueError('temperature 必须在 0-2 之间')
        return v

    @validator('max_tokens')
    def validate_max_tokens(cls, v):
        if not 1 <= v <= 8000:
            raise ValueError('max_tokens 必须在 1-8000 之间')
        return v
```

## 对话系统设计

### 消息格式规范

```python
from enum import Enum
from pydantic import BaseModel

class MessageRole(str, Enum):
    USER = "user"
    ASSISTANT = "assistant"
    SYSTEM = "system"

class ChatMessage(BaseModel):
    role: MessageRole
    content: str
    timestamp: Optional[datetime] = None
    metadata: Optional[Dict[str, Any]] = None

class ChatRequest(BaseModel):
    message: str
    chat_id: Optional[int] = None
    agent_id: Optional[int] = None
    stream: bool = True
```

### 对话历史管理

```python
async def build_conversation_context(
    db: Session,
    chat_id: int,
    max_messages: int = 20
) -> List[Dict[str, str]]:
    # 获取最近的消息历史
    messages = await get_chat_messages(db, chat_id, limit=max_messages)

    # 转换为 OpenAI 格式
    context = []
    for msg in messages:
        context.append({
            "role": msg.role,
            "content": msg.content
        })

    return context

async def save_chat_message(
    db: Session,
    chat_id: int,
    role: MessageRole,
    content: str
) -> Message:
    message = Message(
        chat_id=chat_id,
        role=role.value,
        content=content,
        created_at=datetime.utcnow()
    )

    db.add(message)
    await db.commit()
    await db.refresh(message)

    return message
```

## 任务处理系统

### 任务识别

```python
class TaskClassifier:
    TASK_KEYWORDS = [
        "帮我", "协助", "完成", "处理", "分析", "生成",
        "创建", "制作", "设计", "规划", "解决"
    ]

    def __init__(self):
        self.llm_service = LLMService()

    async def is_task_request(self, user_input: str) -> bool:
        # 简单关键词匹配
        if any(keyword in user_input for keyword in self.TASK_KEYWORDS):
            return True

        # 使用 LLM 进行智能判断
        system_prompt = """
        你是一个任务识别专家。判断用户的输入是否为任务型请求。
        任务型请求包括：需要具体执行的工作、多步骤的操作、创建或生成内容等。
        回答 "是" 或 "否"。
        """

        messages = [
            {"role": "system", "content": system_prompt},
            {"role": "user", "content": f"请判断这是否为任务请求：{user_input}"}
        ]

        response = await self.llm_service.create_chat_completion(
            messages=messages,
            model="gpt-4o-mini",
            max_tokens=10,
            temperature=0.1
        )

        result = response.choices[0].message.content.strip()
        return "是" in result
```

## 错误处理和监控

### API 错误处理

```python
import openai
from tenacity import retry, stop_after_attempt, wait_exponential

class LLMError(Exception):
    pass

class LLMService:
    @retry(
        stop=stop_after_attempt(3),
        wait=wait_exponential(multiplier=1, min=4, max=10)
    )
    async def create_chat_completion_with_retry(self, **kwargs):
        try:
            return await self.client.chat.completions.create(**kwargs)
        except openai.RateLimitError:
            logger.warning("OpenAI API 速率限制，等待重试...")
            raise
        except openai.APIError as e:
            logger.error(f"OpenAI API 错误: {e}")
            raise LLMError(f"AI 服务暂时不可用: {e}")
        except Exception as e:
            logger.error(f"LLM 调用异常: {e}")
            raise LLMError(f"AI 响应生成失败: {e}")
```

### 性能监控

```python
import time
from contextlib import asynccontextmanager

@asynccontextmanager
async def monitor_llm_call(operation: str):
    start_time = time.time()
    try:
        logger.info(f"开始 LLM 操作: {operation}")
        yield
    except Exception as e:
        logger.error(f"LLM 操作失败 {operation}: {e}")
        raise
    finally:
        duration = time.time() - start_time
        logger.info(f"LLM 操作完成 {operation}: {duration:.2f}s")
```

## 开发规范

- **详细注释**: 所有 AI 相关代码必须添加详细的中文注释。
- **异步处理**: 所有 LLM 调用使用 `async/await`。
- **错误处理**: 完善的异常处理和重试机制。
- **流式响应**: 优先使用流式响应提升用户体验。
- **配置管理**: Agent 参数应可配置。
- **监控日志**: 记录关键操作和性能指标。
- **安全措施**: API Key管理、内容过滤、使用量限制。

---
> Source: [open-v2ai/build-ai-template](https://github.com/open-v2ai/build-ai-template) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-06 -->
