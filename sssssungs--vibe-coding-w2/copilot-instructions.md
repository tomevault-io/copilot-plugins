## vibe-coding-w2

> subgraph "Client Layer"

# 시스템 아키텍처 (System Architecture)

## 아키텍처 구성요소

### 전체 시스템 아키텍처
```mermaid
graph TB
    subgraph "Client Layer"
        U[사용자]
        B[웹 브라우저]
    end
    
    subgraph "Frontend Layer"
        ST[Streamlit App]
        UI[챗봇 인터페이스]
        SS[Session State]
    end
    
    subgraph "Backend Layer"
        API[FastAPI Server]
        SSE[SSE Endpoint]
        REST[REST API]
    end
    
    subgraph "Agent Layer"
        LG[LangGraph Agent]
        RA[React Agent]
        PAR[텍스트 파싱]
        VER[정보 검증]
        MEM[Local Memory]
    end
    
    subgraph "LLM Layer"
        GM[Gemini-2.5-flash]
    end
    
    subgraph "External Services"
        NS[Naver Search MCP]
        ES[Exa Search MCP]
        LS[LangSmith]
    end
    
    U --> B
    B --> ST
    ST --> UI
    UI --> SS
    ST <--> API
    API --> SSE
    API --> REST
    API <--> LG
    LG --> RA
    RA --> MEM
    LG --> PAR
    LG --> VER
    PAR --> GM
    VER --> GM
    GM --> PAR
    GM --> VER
    LG <--> NS
    LG <--> ES
    LG --> LS
    
    style U fill:#ff9999
    style ST fill:#99ccff
    style API fill:#99ff99
    style LG fill:#ffcc99
    style GM fill:#cc99ff
```

## 계층 구조

### 5계층 아키텍처
```mermaid
graph TD
    subgraph "Layer 1: Presentation"
        L1[Streamlit 챗봇 UI<br/>- 사용자 입력<br/>- 결과 표시<br/>- 세션 관리]
    end
    
    subgraph "Layer 2: API Gateway"
        L2[FastAPI 서버<br/>- RESTful API<br/>- SSE 스트리밍<br/>- 요청 라우팅]
    end
    
    subgraph "Layer 3: Business Logic"
        L3[LangGraph Agent<br/>- 워크플로우 관리<br/>- 비즈니스 로직<br/>- 상태 관리]
    end
    
    subgraph "Layer 4: AI Processing"
        L4[Gemini LLM<br/>- 텍스트 파싱<br/>- 정보 검증<br/>- 응답 생성]
    end
    
    subgraph "Layer 5: External Data"
        L5[MCP Tools<br/>- 네이버 검색<br/>- Exa 검색<br/>- 웹 크롤링]
    end
    
    L1 --> L2
    L2 --> L3
    L3 --> L4
    L3 --> L5
    L4 --> L3
    L5 --> L3
    L3 --> L2
    L2 --> L1
    
    style L1 fill:#ff9999
    style L2 fill:#99ccff
    style L3 fill:#99ff99
    style L4 fill:#ffcc99
    style L5 fill:#cc99ff
```

## 모듈화 및 컴포넌트

### 컴포넌트 다이어그램
```mermaid
graph TB
    subgraph "Frontend Components"
        CHAT[Chat Interface]
        INPUT[Input Component]
        OUTPUT[Output Component]
        STATE[State Manager]
    end
    
    subgraph "Backend Components"
        ROUTER[API Router]
        STREAM[Stream Handler]
        ERROR[Error Handler]
        MIDDLEWARE[Middleware]
    end
    
    subgraph "Agent Components"
        WORKFLOW[Workflow Engine]
        TOOLS[Tool Manager]
        MEMORY[Memory Manager]
        PARSER[Response Parser]
    end
    
    subgraph "External Components"
        NAVER[Naver MCP]
        EXA[Exa MCP]
        GEMINI[Gemini API]
        LANGSMITH[LangSmith]
    end
    
    CHAT --> INPUT
    CHAT --> OUTPUT
    CHAT --> STATE
    
    ROUTER --> STREAM
    ROUTER --> ERROR
    ROUTER --> MIDDLEWARE
    
    WORKFLOW --> TOOLS
    WORKFLOW --> MEMORY
    WORKFLOW --> PARSER
    
    TOOLS --> NAVER
    TOOLS --> EXA
    PARSER --> GEMINI
    WORKFLOW --> LANGSMITH
    
    STATE <--> ROUTER
    ROUTER <--> WORKFLOW
    
    style CHAT fill:#ff9999
    style ROUTER fill:#99ccff
    style WORKFLOW fill:#99ff99
    style NAVER fill:#ffcc99
```

## 데이터 흐름

### 상품 검색 데이터 플로우
```mermaid
sequenceDiagram
    participant U as 사용자
    participant ST as Streamlit
    participant API as FastAPI
    participant AG as LangGraph Agent
    participant GM as Gemini LLM
    participant NS as Naver Search
    participant ES as Exa Search
    
    U->>ST: 상품명 입력
    ST->>API: POST /search
    API->>AG: 검색 요청
    
    par 병렬 검색
        AG->>NS: 네이버 검색
        AG->>ES: Exa 검색
    end
    
    NS-->>AG: 검색 결과 1
    ES-->>AG: 검색 결과 2
    
    AG->>GM: 결과 파싱 요청
    GM-->>AG: 파싱된 상품 정보
    
    AG->>GM: 가격 비교 분석
    GM-->>AG: 비교 결과
    
    AG-->>API: 최종 결과
    API-->>ST: SSE 스트리밍
    ST-->>U: 가격 비교 표시
    
    Note over U,ES: 실시간 스트리밍으로<br/>진행 상황 표시
```

### 메모리 및 세션 관리
```mermaid
graph LR
    subgraph "LangGraph Memory"
        IMS[InMemorySaver]
        STORE[InMemoryStore]
        CONFIG[Config Management]
    end
    
    subgraph "Session Context"
        TID[Thread ID]
        UID[User ID]
        STATE[Session State]
    end
    
    TID --> CONFIG
    UID --> CONFIG
    CONFIG --> IMS
    CONFIG --> STORE
    IMS --> STATE
    STORE --> STATE
    STATE --> |멀티턴 대화| IMS
    STATE --> |세션 데이터| STORE
    
    style IMS fill:#ff9999
    style STORE fill:#99ccff
    style CONFIG fill:#99ff99
```

## API 및 인터페이스

### API 엔드포인트 구조
```mermaid
graph TB
    subgraph "API Endpoints"
        ROOT["Root /"]
        SEARCH["POST /search"]
        STREAM["GET /stream/{session_id}"]
        HEALTH["GET /health"]
        STATUS["GET /status/{task_id}"]
    end
    
    subgraph "Request/Response"
        REQ[Request Models]
        RES[Response Models]
        SSE_RES[SSE Response]
        ERR[Error Models]
    end
    
    subgraph "Middleware"
        CORS[CORS Middleware]
        LOG[Logging Middleware]
        AUTH[Auth Middleware]
        RATE[Rate Limiting]
    end
    
    ROOT --> REQ
    SEARCH --> REQ
    STREAM --> SSE_RES
    HEALTH --> RES
    STATUS --> RES
    
    REQ --> CORS
    RES --> CORS
    SSE_RES --> LOG
    ERR --> AUTH
    
    CORS --> LOG
    LOG --> AUTH
    AUTH --> RATE
    
    style SEARCH fill:#ff9999
    style STREAM fill:#99ccff
    style SSE_RES fill:#99ff99
```

### 인터페이스 정의
```mermaid
graph LR
    subgraph "Frontend Interface"
        FI[Streamlit Interface]
        HTTP[HTTP Client]
        WS[WebSocket/SSE]
    end
    
    subgraph "Backend Interface"
        BI[FastAPI Interface]
        ASYNC[Async Handler]
        JSON[JSON Response]
    end
    
    subgraph "Agent Interface"
        AI[Agent Interface]
        TOOL[Tool Interface]
        LLM[LLM Interface]
    end
    
    FI --> HTTP
    HTTP --> BI
    BI --> ASYNC
    ASYNC --> AI
    AI --> TOOL
    AI --> LLM
    
    WS --> JSON
    JSON --> FI
    
    style FI fill:#ff9999
    style BI fill:#99ccff
    style AI fill:#99ff99
```

## 시스템 외부 환경과의 관계

### 외부 시스템 연동
```mermaid
graph TB
    subgraph "Internal System"
        APP[Shopping Agent App]
        AGENT[LangGraph Agent]
        API[FastAPI Server]
    end
    
    subgraph "External APIs"
        NAVER[Naver Search API]
        EXA[Exa Search API]
        GEMINI[Google Gemini API]
    end
    
    subgraph "Monitoring & Observability"
        LANGSMITH[LangSmith Platform]
    end
    
    subgraph "Infrastructure"
        WEB[Web Browser]
        SERVER[Application Server]
        NETWORK[Network Layer]
    end
    
    APP --> AGENT
    AGENT --> API
    AGENT --> NAVER
    AGENT --> EXA
    AGENT --> GEMINI
    AGENT --> LANGSMITH
    
    WEB --> APP
    APP --> SERVER
    SERVER --> NETWORK
    NETWORK --> NAVER
    NETWORK --> EXA
    NETWORK --> GEMINI
    
    style APP fill:#ff9999
    style NAVER fill:#99ccff
    style LANGSMITH fill:#99ff99
    style WEB fill:#ffcc99
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/sssssungs)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/sssssungs)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
