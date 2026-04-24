## stock-trading

> Clean Architecture 패턴 및 레이어 구조


# Clean Architecture 구조

프로젝트는 Clean Architecture 패턴을 따릅니다.

## 레이어 구조

```
app/
├── domain/              # 도메인 계층 (의존성 없음)
│   ├── entities/        # 엔티티 정의
│   └── repositories/    # Repository 인터페이스
├── application/         # 애플리케이션 계층
│   ├── use_cases/      # Use Cases (비즈니스 로직)
│   └── dependencies.py # 의존성 주입 설정
├── infrastructure/      # 인프라 계층
│   ├── database/       # DB 클라이언트 (MongoDB, Supabase)
│   └── repositories/   # Repository 구현체
└── api/                # 프레젠테이션 계층
    └── routes/         # FastAPI 라우터
```

## 레이어별 역할

### 1. Domain Layer (도메인 계층)
**위치**: `app/domain/`
- **entities/**: 도메인 엔티티 정의 (의존성 없음)
- **repositories/**: Repository 인터페이스 정의

### 2. Application Layer (애플리케이션 계층)
**위치**: `app/application/`
- **use_cases/**: 비즈니스 로직 구현
- **dependencies.py**: Repository 팩토리 함수 제공

### 3. Infrastructure Layer (인프라 계층)
**위치**: `app/infrastructure/`
- **database/**: MongoDB, Supabase 클라이언트
- **repositories/**: Repository 인터페이스 구현체

### 4. Presentation Layer (프레젠테이션 계층)
**위치**: `app/api/`
- **routes/**: FastAPI 라우터 정의
- **api.py**: 모든 라우터 중앙 등록

## 개발 워크플로우

1. **도메인 정의**: `domain/entities/`에 엔티티 정의
2. **Repository 인터페이스**: `domain/repositories/`에 인터페이스 정의
3. **Repository 구현**: `infrastructure/repositories/`에 구현
4. **Use Case 구현**: `application/use_cases/`에 비즈니스 로직 구현
5. **API 라우터**: `api/routes/`에 엔드포인트 정의
6. **의존성 주입**: `application/dependencies.py`에서 Repository 팩토리 함수 제공

## 새 기능 추가 시

1. 도메인 엔티티 정의 (`domain/entities/`)
2. Repository 인터페이스 정의 (`domain/repositories/`)
3. Repository 구현 (`infrastructure/repositories/`)
4. Use Case 구현 (`application/use_cases/`)
5. API 라우터 추가 (`api/routes/`)
6. `api/api.py`에 라우터 등록

## 의존성 주입

`app/application/dependencies.py`에서 Repository 팩토리 함수 제공:

```python
from app.application.dependencies import get_stock_repository
repository = get_stock_repository()  # MongoDB Repository 반환
```

**⚠️ 중요**: Repository 팩토리 함수는 조회 시 MongoDB 구현체를 반환해야 합니다.

## 의존성 방향

```
Presentation → Application → Domain
                ↓
           Infrastructure
```

## 코드 예시

### Use Case
```python
from app.domain.repositories.stock_repository import StockRepository
from app.application.dependencies import get_stock_repository

class GetStockRecommendationsUseCase:
    def __init__(self):
        self.repository: StockRepository = get_stock_repository()
    
    async def execute(self) -> List[StockRecommendation]:
        return await self.repository.find_recommendations()
```

### API Router
```python
from fastapi import APIRouter, Depends
from app.application.use_cases.get_stock_recommendations import GetStockRecommendationsUseCase

router = APIRouter(prefix="/stocks", tags=["stocks"])

@router.get("/recommendations")
async def get_recommendations(
    use_case: GetStockRecommendationsUseCase = Depends()
):
    return await use_case.execute()
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/lian220) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
