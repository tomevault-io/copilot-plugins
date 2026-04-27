## bok-compensation-regulations

> 한국은행 보수규정 하이브리드 AI 에이전트 시스템. TypeDB/Neo4j 지식그래프 + Context RAG + ReAct 에이전트를 결합하여 HR 보수규정 질의응답을 수행한다.

# CLAUDE.md

## Project Overview
한국은행 보수규정 하이브리드 AI 에이전트 시스템. TypeDB/Neo4j 지식그래프 + Context RAG + ReAct 에이전트를 결합하여 HR 보수규정 질의응답을 수행한다.

## Architecture
- **3 Backend**: TypeDB KG RAG, Neo4j Graph RAG, Context-only RAG
- **MoE**: HCX (메인 추론) + Qwen (DB 쿼리 전문)
- **Agent Framework**: LangGraph (ReAct + Reflection)
- **UI**: Streamlit (`app.py`)
- **hybrid_router_graph.py**: 현재 시뮬레이션 코드만 존재 (실제 DB 미연동)

## Directory Structure
```
src/
  bok_compensation_typedb/  # TypeDB backend (agent, insert_data, schema, config)
  bok_compensation_neo4j/   # Neo4j backend (agent, insert_data, config)
  bok_compensation_context/ # Context RAG (context_query, regulation_context.md)
  bok_compensation/         # hybrid_router_graph (simulation only)
schema/                     # TypeDB .tql schema
tests/                      # pytest tests
docs/                       # Documentation, PDF source
```

## Key Commands
```bash
# Setup
chmod +x setup.sh && ./setup.sh

# Run
source .venv/bin/activate
streamlit run app.py

# Test
PYTHONPATH=src pytest tests/

# Data validation
PYTHONPATH=src python tests/validate_data.py typedb
```

## LLM Configuration
- `llm_template.py`: 커밋된 LLM 팩토리 (env 기반)
- `llm.py`: gitignore된 실제 설정 파일 (민감 정보 포함)
- 환경변수: `.env` 파일 사용 (`.env.example` 참조)

## Rules
- `llm.py`는 절대 커밋하지 말 것 (API 키 포함)
- DB 수치 변경 시 `tests/validate_data.py`로 PDF 원본 대비 검증 필수
- TypeQL 3.x 문법 사용 (`get`, `return` 키워드 없음, `match` 블록만)
- 한국어 엔티티명 사용 (직급, 직위, 호봉 등)
- 코드 내 규정 텍스트는 `regulation_context.md`에 전처리된 형태로 관리

## Known Issues
- `tests/test_intent_extraction.py`가 삭제된 `nl_query` 모듈을 참조 (테스트 깨짐)
- Neo4j config가 `NEO4J_USER`를 읽지만 `.env.example`은 `NEO4J_USERNAME` 사용
- TypeDB/Neo4j agent의 `build_*_agent()`가 LLM 엔드포인트를 하드코딩
- DB driver close가 finally/context manager 없이 try 내부에서만 호출

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/shezgone) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
