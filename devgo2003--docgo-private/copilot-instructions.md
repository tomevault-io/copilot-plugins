## docgo-private

> Quy tắc chuẩn hóa event payload cho các microservices

# Chuẩn hóa Event Payload Standard

Tất cả events giữa các services phải tuân thủ schema chuẩn sau để đảm bảo tính nhất quán và traceability:

## Schema Event Payload Chuẩn

```json
{
  "eventVersion": "<string>",
  "eventType": "<string>",
  "eventId": "<uuid>",
  "timestamp": "<ISO-8601>",
  "source": "<string>",
  "correlationId": "<uuid>",
  "actor": {
    "userId": "<string>",
    "userRole": "<string>",
    "ip": "<string>"
  },
  "data": {
    /* cấu trúc thay đổi tùy eventType */
  },
  "metadata": {
    "region": "<string>",
    "serviceVersion": "<string>"
  }
}
```

## Quy tắc Event Types

### File Events
- `FileUploaded` - File được upload thành công
- `FileProcessed` - File được xử lý hoàn tất
- `FileDeleted` - File bị xóa
- `FileUpdated` - File được cập nhật

### User Events
- `UserLoggedIn` - Người dùng đăng nhập
- `UserCreated` - Người dùng được tạo mới
- `UserUpdated` - Thông tin người dùng được cập nhật
- `UserDeleted` - Người dùng bị xóa

### Document Events
- `DocumentCreated` - Tài liệu được tạo mới
- `DocumentUpdated` - Tài liệu được cập nhật
- `DocumentDeleted` - Tài liệu bị xóa
- `DocumentProcessed` - Tài liệu được xử lý

### Automation Events
- `AutomationStarted` - Quá trình automation bắt đầu
- `AutomationCompleted` - Quá trình automation hoàn thành
- `AutomationFailed` - Quá trình automation thất bại

## Implementation

### Java (Spring)
```java
@Component
public class EventPublisher {
    
    @Autowired
    private ApplicationEventPublisher eventPublisher;
    
    public void publishEvent(String eventType, Object data, String correlationId) {
        EventPayload event = EventPayload.builder()
            .eventVersion("v1")
            .eventType(eventType)
            .eventId(UUID.randomUUID().toString())
            .timestamp(Instant.now().toString())
            .source("user-management-service")
            .correlationId(correlationId)
            .actor(Actor.builder()
                .userId(getCurrentUserId())
                .userRole(getCurrentUserRole())
                .ip(getClientIp())
                .build())
            .data(data)
            .metadata(Metadata.builder()
                .region("ap-southeast-1")
                .serviceVersion("1.0.0")
                .build())
            .build();
            
        eventPublisher.publishEvent(event);
    }
}
```

### Python (FastAPI)
```python
import redis
import json
from datetime import datetime
from uuid import uuid4

class EventPublisher:
    def __init__(self):
        self.redis_client = redis.Redis(host='localhost', port=6379, db=0)
    
    def publish_event(self, event_type: str, data: dict, correlation_id: str):
        event = {
            "eventVersion": "v1",
            "eventType": event_type,
            "eventId": str(uuid4()),
            "timestamp": datetime.utcnow().isoformat() + "Z",
            "source": "automation-service",
            "correlationId": correlation_id,
            "actor": {
                "userId": self.get_current_user_id(),
                "userRole": self.get_current_user_role(),
                "ip": self.get_client_ip()
            },
            "data": data,
            "metadata": {
                "region": "ap-southeast-1",
                "serviceVersion": "1.0.0"
            }
        }
        
        # Publish to Redis channel
        self.redis_client.publish("events", json.dumps(event))
```

## Message Broker Configuration

### Redis Pub/Sub
```python
# Publisher
redis_client.publish("file.events", json.dumps(event_payload))

# Subscriber
pubsub = redis_client.pubsub()
pubsub.subscribe("file.events")
for message in pubsub.listen():
    if message['type'] == 'message':
        event = json.loads(message['data'])
        process_event(event)
```

### Apache Kafka
```java
@KafkaListener(topics = "file.events")
public void handleFileEvent(EventPayload event) {
    log.info("Received event: {}", event.getEventType());
    // Process event
}
```

## Correlation ID Management

### Tự động generate từ request context
```java
@Component
public class CorrelationIdFilter implements Filter {
    
    @Override
    public void doFilter(ServletRequest request, ServletResponse response, 
                        FilterChain chain) throws IOException, ServletException {
        
        HttpServletRequest httpRequest = (HttpServletRequest) request;
        String correlationId = httpRequest.getHeader("X-Correlation-ID");
        
        if (correlationId == null) {
            correlationId = UUID.randomUUID().toString();
        }
        
        MDC.put("correlationId", correlationId);
        
        try {
            chain.doFilter(request, response);
        } finally {
            MDC.remove("correlationId");
        }
    }
}
```

### Truyền từ upstream
```python
def process_request(request: Request):
    correlation_id = request.headers.get("X-Correlation-ID")
    if not correlation_id:
        correlation_id = str(uuid4())
    
    # Pass correlation_id to all downstream calls
    publish_event("RequestProcessed", data, correlation_id)
```

## Event Schema Validation

### Java (Spring Boot)
```java
@JsonSchema(description = "Event payload schema")
public class EventPayload {
    @NotBlank
    private String eventVersion;
    
    @NotBlank
    private String eventType;
    
    @NotBlank
    private String eventId;
    
    @NotBlank
    private String timestamp;
    
    @NotBlank
    private String source;
    
    @NotBlank
    private String correlationId;
    
    @Valid
    private Actor actor;
    
    private Object data;
    
    @Valid
    private Metadata metadata;
}
```

### Python (FastAPI)
```python
from pydantic import BaseModel, Field
from typing import Optional, Any
from datetime import datetime

class Actor(BaseModel):
    userId: Optional[str] = None
    userRole: Optional[str] = None
    ip: Optional[str] = None

class Metadata(BaseModel):
    region: str
    serviceVersion: str

class EventPayload(BaseModel):
    eventVersion: str = Field(..., description="Event schema version")
    eventType: str = Field(..., description="Type of event")
    eventId: str = Field(..., description="Unique event identifier")
    timestamp: str = Field(..., description="Event timestamp in ISO-8601")
    source: str = Field(..., description="Source service name")
    correlationId: str = Field(..., description="Correlation ID for tracing")
    actor: Actor = Field(..., description="Actor information")
    data: Optional[Any] = Field(None, description="Event data payload")
    metadata: Metadata = Field(..., description="Event metadata")
```

---

**Lưu ý**: Tất cả events giữa các services phải tuân thủ schema này để đảm bảo tính nhất quán và khả năng trace.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/DevGO2003) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
