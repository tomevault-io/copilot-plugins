## vibecoding-2-2

> tech stack

# 기술 스택 (Tech Stack)

## 프론트엔드 (Frontend)

### Streamlit
- **언어**: Python 3.11
- **프레임워크**: Streamlit
- **용도**: 챗봇 인터페이스 구현
- **주요 기능**:
  - 상품 검색 결과 표시


## 백엔드 (Backend)

### FastAPI
- **언어**: Python 3.11
- **프레임워크**: FastAPI
- **용도**: API 서버 및 Agent 연동
- **주요 기능**:
  - RESTful API 엔드포인트
  - Agent 호출 및 결과 처리

### 주요 라이브러리
- **FastAPI**: 웹 프레임워크
- **uvicorn**: ASGI 서버

## Agent

### LangGraph
- **프레임워크**: LangGraph
- **Agent 타입**: Pre-built React Agent, single turn
- **모니터링**: LangSmith 연동, .env에 LANGSMITH_API_KEY 설정

### LLM
- **모델**: Gemini-2.5-flash-preview-05-20
- **용도**: 
  - React Agent로 덕덕고 Tool로 인터넷 검색해서 결과를 받아서 처리

### Tools
- **LangChain DuckDuckGo**: LangChain의 DuckDuckGo Search Tool
- **용도**: 웹 검색

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jibongPark) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
