## storybuilder

> Backend Hybrid·FastAPI·도메인 경계


# Backend 규칙

## 스택·구조
- FastAPI. 도메인별 라우터: `api/projects.py`, `chapters.py`, `scenes.py`, `manuscripts.py`, `characters.py` 등.
- 향후 분리: API Gateway / Story Engine(FastAPI) / AI Engine(FastAPI). 새 기능은 도메인 경계에 맞춰 배치.

## DB·캐시
- PostgreSQL: Core 엔티티 (User, Project, Chapter, Scene, Manuscript, Character, Timeline, Setting). SQLAlchemy + Alembic.
- Redis: 실시간 저장·자동저장·세션. WebSocket 연동 시 Redis 사용.

## 컨벤션
- 엔드포인트는 REST, 스키마는 Pydantic. ERD(`docs/erd.md`) 및 데이터 딕셔너리와 불일치 없이 모델 추가.
- 신규 API는 `api/` 하위 모듈로 추가하고 `main.py`에 라우터 등록.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/chenluc1a)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/chenluc1a)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
