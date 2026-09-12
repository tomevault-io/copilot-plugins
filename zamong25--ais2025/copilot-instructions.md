## ais2025

> > 이 문서는 AI(Claude)가 델파이 트레이더 프로젝트를 일관성 있게 개발하기 위한 가이드입니다.

# Claude 작업 가이드 v2.0

> 이 문서는 AI(Claude)가 델파이 트레이더 프로젝트를 일관성 있게 개발하기 위한 가이드입니다.

## 🚀 세션 시작 프로토콜

### 0. 필수 정보 확인 ⚠️
```bash
cat .ai/CRITICAL_INFO.md          # 실행 환경, 주의사항 확인 (필독!)
```

### 1. 상태 확인 (필수)
```bash
# 순서대로 실행
1. cat .ai/CURRENT_STATE.yaml     # 현재 상태 파악
2. cat .ai/WORK_LOG.yaml          # 마지막 작업 확인  
3. ls .ai/workspace/              # 임시 파일 확인
```

### 2. 작업 준비
```bash
# 작업 공간 정리
rm -f .ai/workspace/test_*.py
rm -f .ai/workspace/analyze_*.py

# 오늘 날짜로 새 세션 시작
SESSION_ID=$(date +%Y-%m-%d)-001
```

### 3. 의도 확인
- 작업 전 `intentions/` 폴더의 관련 문서 확인
- 코드 작성보다 의도 이해가 우선

## 📝 코딩 규칙

### 1. 함수 설계
```python
# ❌ 나쁜 예 - 여러 책임
def process_trade(data):
    # 검증
    if not data.get('symbol'):
        return False
    
    # 계산
    size = calculate_size(data)
    
    # DB 저장
    save_to_db(data)
    
    # 알림
    send_notification(data)
    
    return True

# ✅ 좋은 예 - 단일 책임
def validate_trade_data(data: Dict[str, Any]) -> bool:
    """거래 데이터 검증만 수행"""
    required_fields = ['symbol', 'quantity', 'price']
    return all(field in data for field in required_fields)

def calculate_position_size(capital: float, risk: float) -> float:
    """포지션 크기 계산만 수행"""
    return capital * risk / 100
```

### 2. 설정 관리
```python
# ❌ 하드코딩
leverage = 10
symbol = "SOLUSDT"

# ✅ 설정 파일 사용
from config import settings

leverage = settings.trading.default_leverage
symbol = settings.trading.symbol
```

### 3. 에러 처리
```python
# ❌ 모호한 에러
except Exception as e:
    logger.error("에러 발생")

# ✅ 구체적인 에러
except ConnectionError as e:
    logger.error(f"거래소 연결 실패: {e}")
    raise ExchangeConnectionError(f"바이낸스 연결 실패: {e}")
```

### 4. 타입 힌트
```python
# ❌ 타입 힌트 없음
def calculate_pnl(entry, exit, quantity):
    return (exit - entry) * quantity

# ✅ 명확한 타입 힌트
from decimal import Decimal

def calculate_pnl(
    entry_price: Decimal,
    exit_price: Decimal,
    quantity: Decimal
) -> Decimal:
    """손익 계산 (타입 안전)"""
    return (exit_price - entry_price) * quantity
```

## 🏗️ 아키텍처 규칙

### 1. 계층 분리
```
도메인(Domain) → 순수 비즈니스 로직, 외부 의존성 없음
    ↓
애플리케이션(Application) → 비즈니스 유스케이스, 오케스트레이션
    ↓  
인프라(Infrastructure) → 외부 시스템 연동 (DB, API, 파일 등)
    ↓
프레젠테이션(Presentation) → 사용자 인터페이스 (CLI, API, 스케줄러)
```

### 2. 의존성 규칙
- 안쪽 계층은 바깥쪽을 모름
- 인터페이스로 의존성 역전
- 구체적 구현은 인프라에만

### 3. 파일 위치
```python
# 도메인 모델 → domain/trading/models/
class Position:
    """거래 포지션"""

# 비즈니스 규칙 → domain/trading/rules/
def should_close_position(position: Position) -> bool:
    """포지션 청산 여부 결정"""

# 서비스 → application/services/
class TradingService:
    """거래 오케스트레이션"""

# 외부 연동 → infrastructure/exchanges/
class BinanceClient:
    """바이낸스 API 구현"""
```

## 🧪 테스트 작성

### 1. 테스트 위치
```bash
# 임시 테스트 → .ai/workspace/
.ai/workspace/test_new_feature.py  # 작업 후 삭제

# 영구 테스트 → tests/
tests/unit/domain/test_position.py
tests/integration/test_trading_service.py
```

### 2. 테스트 구조
```python
# tests/unit/domain/test_position.py
import pytest
from decimal import Decimal
from domain.trading.models import Position

class TestPosition:
    def test_calculate_pnl_long(self):
        # Given
        position = Position(
            symbol="SOLUSDT",
            direction="LONG",
            entry_price=Decimal("100"),
            quantity=Decimal("10")
        )
        
        # When
        pnl = position.calculate_pnl(Decimal("110"))
        
        # Then
        assert pnl == Decimal("100")  # (110-100)*10
```

## 📊 상태 관리

### 1. 작업 시작 시
```yaml
# .ai/CURRENT_STATE.yaml 업데이트
current_work:
  tasks:
    in_progress:
      - "Position 모델 구현"
```

### 2. 작업 중
- 발견한 문제는 즉시 기록
- 중요한 결정은 이유와 함께 기록

### 3. 작업 종료 시
```yaml
# .ai/WORK_LOG.yaml에 세션 추가
sessions:
  - session_id: "2024-01-20-001"
    tasks_completed:
      - "Position 모델 구현"
    files_created:
      - "domain/trading/models/position.py"
    next_session_tasks:
      - "Position 테스트 작성"
```

## 🚫 절대 하지 말 것

1. **한 번에 큰 변경**
   - legacy/ 코드 전체 리팩토링 ❌
   - 하나씩 점진적 마이그레이션 ✅

2. **테스트 없는 리팩토링**
   - 기능 변경 후 바로 커밋 ❌
   - 테스트 작성 → 리팩토링 → 테스트 통과 ✅

3. **문서 업데이트 누락**
   - 코드만 변경 ❌
   - 코드 + 의도 문서 + 상태 파일 ✅

4. **하드코딩**
   - 값을 코드에 직접 입력 ❌
   - config/ 파일 사용 ✅

5. **긴 함수**
   - 100줄 함수 ❌
   - 30줄 이하로 분리 ✅

## 🎯 우선순위

### 긴급 (이번 주)
1. 포지션 캐싱 버그 수정
2. 설정 파일 추출
3. 핵심 도메인 모델 구현

### 중요 (이번 달)
1. 거래 시스템 마이그레이션
2. 테스트 인프라 구축
3. AI 에이전트 리팩토링

### 장기 (3개월)
1. 완전한 클린 아키텍처 전환
2. 테스트 커버리지 80%
3. 성능 최적화

## 💡 유용한 스니펫

### 새 도메인 모델
```python
"""[모델명] 도메인 모델"""
from dataclasses import dataclass
from typing import Optional

@dataclass
class ModelName:
    """[설명]"""
    # 필수 필드
    field1: type
    
    # 선택 필드  
    field2: Optional[type] = None
    
    def business_method(self) -> type:
        """비즈니스 로직"""
        pass
```

### 새 서비스
```python
"""[서비스명] 애플리케이션 서비스"""
from application.interfaces import IInterface

class ServiceName:
    """[설명]"""
    
    def __init__(self, dependency: IInterface):
        self.dependency = dependency
    
    def use_case_method(self) -> Result:
        """유스케이스 구현"""
        # 1. 검증
        # 2. 비즈니스 로직
        # 3. 영속성
        # 4. 응답
        pass
```

### 설정 로드
```python
"""설정 로더"""
import yaml
from pathlib import Path

def load_config(env: str = "development") -> dict:
    base = Path("config/base")
    env_config = Path(f"config/environments/{env}.yaml")
    
    # 기본 설정 로드
    config = {}
    for file in base.glob("*.yaml"):
        with open(file) as f:
            config.update(yaml.safe_load(f))
    
    # 환경별 설정 오버라이드
    if env_config.exists():
        with open(env_config) as f:
            config.update(yaml.safe_load(f))
    
    return config
```

## 📞 도움이 필요할 때

1. **의도 불명확**: `intentions/` 문서 확인
2. **이전 작업 참고**: `.ai/WORK_LOG.yaml` 검색
3. **아키텍처 의문**: `docs/SYSTEM_REDESIGN_MASTER_PLAN.md` 참조

---

**"의도를 코드로, 복잡함을 단순하게"**

버전: 2.0.0
최종 수정: 2024-01-20

---
> Source: [zamong25/AIS2025](https://github.com/zamong25/AIS2025) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-11 -->
