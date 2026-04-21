## weather-flick-batch

> Weather Flick Batch System Development Rules


# Weather Flick Batch System Team Rules

당신은 Weather Flick 배치 시스템 개발의 전문가입니다. Python, APScheduler, 대용량 데이터 처리, 외부 API 연동, 그리고 배치 시스템 최적화에 능숙합니다.

## 🎯 프로젝트 컨텍스트

### 역할 및 책임
- **데이터 수집 시스템**: KTO (한국관광공사), KMA (기상청) API 연동
- **배치 스케줄링**: APScheduler 기반 정기 작업 실행
- **데이터 처리 파이프라인**: 수집 → 변환 → 검증 → 저장
- **추천 엔진**: 날씨 기반 여행지 추천 점수 계산
- **시스템 유지보수**: 백업, 로그 관리, 데이터 품질 검사

### 기술 스택
- **Framework**: APScheduler + Python 3.11+
- **API 클라이언트**: 다중 API 키 관리 시스템
- **Data Processing**: Pandas, NumPy 기반 대용량 처리
- **Database**: PostgreSQL + SQLAlchemy
- **Caching**: Redis 기반 캐싱 시스템
- **Code Quality**: Ruff + Black + MyPy + pytest

## 📁 필수 프로젝트 구조

```
app/
├── core/                    # 핵심 시스템 모듈
│   ├── base_job.py         # 배치 작업 기본 클래스
│   ├── multi_api_key_manager.py # API 키 관리 시스템
│   ├── smart_scheduler.py  # 스마트 스케줄러
│   ├── error_handling.py   # 통합 에러 처리
│   └── logger.py          # 로깅 시스템
├── collectors/             # 데이터 수집 모듈
│   ├── kto_collector.py    # KTO API 수집기
│   ├── weather_collector.py # 기상청 API 수집기
│   └── unified_kto_client.py # 통합 KTO 클라이언트
├── processors/            # 데이터 처리 모듈
│   ├── data_transformation_pipeline.py
│   └── tourism_data_processor.py
├── jobs/                 # 배치 작업 정의
│   ├── weather/          # 날씨 데이터 작업
│   ├── tourism/          # 관광 데이터 작업
│   ├── recommendation/   # 추천 시스템 작업
│   ├── quality/         # 데이터 품질 검사
│   └── system_maintenance/ # 시스템 유지보수
└── schedulers/          # 스케줄러 관리
    └── advanced_scheduler.py
```

## 🛠️ 핵심 개발 원칙

### 1. 배치 작업 기본 패턴 (필수 준수)

```python
from abc import ABC, abstractmethod
from typing import Dict, Any, Optional
from datetime import datetime
import logging

from app.core.error_handling import handle_batch_error, BatchRetryStrategy
from app.core.logger import get_batch_logger

class BaseJob(ABC):
    """배치 작업 기본 클래스"""
    
    def __init__(self, job_name: str):
        self.job_name = job_name
        self.logger = get_batch_logger(job_name)
        self.start_time: Optional[datetime] = None
        self.retry_strategy = BatchRetryStrategy()
    
    async def execute(self, **kwargs) -> Dict[str, Any]:
        """
        배치 작업 실행 메인 메서드
        
        Returns:
            Dict[str, Any]: 작업 실행 결과
        
        Raises:
            BatchJobError: 배치 작업 실행 실패
        """
        self.start_time = datetime.now()
        job_context = {
            "job_name": self.job_name,
            "start_time": self.start_time,
            "parameters": kwargs
        }
        
        try:
            self.logger.info(f"배치 작업 시작: {self.job_name}")
            
            # Pre-execution validation
            await self.validate_prerequisites()
            
            # Main job execution
            result = await self.run_job(**kwargs)
            
            # Post-execution cleanup
            await self.cleanup()
            
            execution_time = (datetime.now() - self.start_time).total_seconds()
            success_result = {
                "status": "SUCCESS",
                "execution_time": execution_time,
                "processed_records": result.get("processed_records", 0),
                "result": result
            }
            
            self.logger.info(
                f"배치 작업 완료: {self.job_name}, "
                f"실행시간: {execution_time:.2f}초, "
                f"처리 레코드: {result.get('processed_records', 0)}개"
            )
            
            return success_result
            
        except Exception as e:
            execution_time = (datetime.now() - self.start_time).total_seconds()
            error_result = await handle_batch_error(
                e, job_context, self.retry_strategy
            )
            
            self.logger.error(
                f"배치 작업 실패: {self.job_name}, "
                f"실행시간: {execution_time:.2f}초, "
                f"오류: {str(e)}"
            )
            
            return {
                "status": "FAILED",
                "execution_time": execution_time,
                "error": str(e),
                "error_details": error_result
            }
    
    @abstractmethod
    async def run_job(self, **kwargs) -> Dict[str, Any]:
        """실제 배치 작업 로직 구현"""
        pass
    
    async def validate_prerequisites(self) -> None:
        """작업 실행 전 필수 조건 검증"""
        pass
    
    async def cleanup(self) -> None:
        """작업 완료 후 정리 작업"""
        pass

# 구체적인 배치 작업 구현 예시
class WeatherDataJob(BaseJob):
    """날씨 데이터 수집 배치 작업"""
    
    def __init__(self):
        super().__init__("weather_data_collection")
        self.weather_collector = WeatherCollector()
        self.data_validator = WeatherDataValidator()
    
    async def validate_prerequisites(self) -> None:
        """API 키 및 데이터베이스 연결 확인"""
        if not await self.weather_collector.check_api_availability():
            raise ValueError("기상청 API 연결 불가능")
    
    async def run_job(self, regions: List[str] = None) -> Dict[str, Any]:
        """날씨 데이터 수집 실행"""
        # 입력 검증 (Early Return)
        if not regions:
            regions = await self._get_default_regions()
        
        if not regions:
            raise ValueError("수집할 지역이 없습니다")
        
        total_processed = 0
        failed_regions = []
        
        for region in regions:
            try:
                # 지역별 데이터 수집
                weather_data = await self.weather_collector.collect_weather_data(region)
                
                # 데이터 검증
                validated_data = await self.data_validator.validate(weather_data)
                
                # 데이터베이스 저장
                saved_count = await self._save_weather_data(validated_data)
                total_processed += saved_count
                
                self.logger.info(f"지역 {region} 날씨 데이터 수집 완료: {saved_count}개")
                
            except Exception as e:
                failed_regions.append({"region": region, "error": str(e)})
                self.logger.error(f"지역 {region} 날씨 데이터 수집 실패: {e}")
        
        return {
            "processed_records": total_processed,
            "successful_regions": len(regions) - len(failed_regions),
            "failed_regions": failed_regions,
            "total_regions": len(regions)
        }
```

### 2. 다중 API 키 관리 패턴 (필수)

```python
from typing import List, Dict, Optional, Tuple
import asyncio
import json
from datetime import datetime, timedelta
import redis
import logging

logger = logging.getLogger(__name__)

class MultiApiKeyManager:
    """다중 API 키 관리 시스템"""
    
    def __init__(self, 
                 api_keys: List[str], 
                 daily_limit: int,
                 redis_client: redis.Redis = None):
        self.api_keys = api_keys
        self.daily_limit = daily_limit
        self.redis = redis_client or redis.Redis(host='localhost', port=6379, db=0)
        self.cache_prefix = "api_key_usage"
        self.current_key_index = 0
        
    async def get_available_api_key(self) -> Optional[Tuple[str, int]]:
        """
        사용 가능한 API 키 반환
        
        Returns:
            Tuple[str, int]: (API 키, 남은 사용량) 또는 None
        
        Raises:
            APIKeyExhaustedException: 모든 키 사용량 초과
        """
        for attempt in range(len(self.api_keys)):
            key_index = (self.current_key_index + attempt) % len(self.api_keys)
            api_key = self.api_keys[key_index]
            
            # 키별 사용량 확인
            usage_count = await self._get_key_usage(api_key)
            remaining = self.daily_limit - usage_count
            
            if remaining > 0:
                # 키 인덱스 업데이트 (라운드 로빈)
                self.current_key_index = key_index
                return api_key, remaining
            
            logger.warning(f"API 키 {key_index} 일일 한도 초과: {usage_count}/{self.daily_limit}")
        
        # 모든 키가 한도 초과인 경우
        raise APIKeyExhaustedException(
            f"모든 API 키의 일일 사용량 초과 (총 {len(self.api_keys)}개 키)"
        )
    
    async def record_api_usage(self, api_key: str, request_count: int = 1) -> int:
        """
        API 사용량 기록
        
        Args:
            api_key: 사용된 API 키
            request_count: 요청 수
        
        Returns:
            int: 업데이트된 총 사용량
        """
        cache_key = f"{self.cache_prefix}:{api_key}:{datetime.now().strftime('%Y-%m-%d')}"
        
        try:
            # Redis에 사용량 증가
            current_usage = self.redis.incr(cache_key, request_count)
            
            # TTL 설정 (다음날 자정까지)
            tomorrow = datetime.now().replace(hour=0, minute=0, second=0, microsecond=0) + timedelta(days=1)
            ttl = int((tomorrow - datetime.now()).total_seconds())
            self.redis.expire(cache_key, ttl)
            
            # 사용량 로그
            remaining = self.daily_limit - current_usage
            logger.info(f"API 키 사용량 기록: {current_usage}/{self.daily_limit} (남은 사용량: {remaining})")
            
            # 90% 도달 시 경고
            if current_usage >= self.daily_limit * 0.9:
                logger.warning(f"API 키 사용량 90% 도달: {current_usage}/{self.daily_limit}")
            
            return current_usage
            
        except Exception as e:
            logger.error(f"API 사용량 기록 실패: {e}")
            return 0
    
    async def _get_key_usage(self, api_key: str) -> int:
        """API 키의 일일 사용량 조회"""
        cache_key = f"{self.cache_prefix}:{api_key}:{datetime.now().strftime('%Y-%m-%d')}"
        
        try:
            usage = self.redis.get(cache_key)
            return int(usage) if usage else 0
        except Exception as e:
            logger.error(f"API 사용량 조회 실패: {e}")
            return 0

class APIKeyExhaustedException(Exception):
    """모든 API 키 사용량 초과 예외"""
    pass
```

### 3. 대용량 데이터 처리 패턴

```python
import pandas as pd
import numpy as np
from typing import Iterator, Dict, List, Any
import logging
from concurrent.futures import ThreadPoolExecutor
import asyncio

logger = logging.getLogger(__name__)

class BatchDataProcessor:
    """대용량 데이터 배치 처리기"""
    
    def __init__(self, 
                 chunk_size: int = 10000,
                 max_workers: int = 4,
                 memory_limit_mb: int = 512):
        self.chunk_size = chunk_size
        self.max_workers = max_workers
        self.memory_limit_bytes = memory_limit_mb * 1024 * 1024
    
    async def process_large_dataset(self, 
                                   data_source: Any,
                                   processor_func: callable,
                                   **kwargs) -> Dict[str, Any]:
        """
        대용량 데이터셋 청크 단위 처리
        
        Args:
            data_source: 데이터 소스 (파일, DB 쿼리 등)
            processor_func: 청크 처리 함수
            **kwargs: 처리 함수에 전달할 추가 인자
        
        Returns:
            Dict[str, Any]: 처리 결과 통계
        """
        total_processed = 0
        total_chunks = 0
        failed_chunks = 0
        processing_errors = []
        
        try:
            # 메모리 사용량 모니터링
            import psutil
            process = psutil.Process()
            initial_memory = process.memory_info().rss
            
            # 청크 단위로 데이터 처리
            async for chunk_data in self._get_data_chunks(data_source):
                total_chunks += 1
                
                try:
                    # 메모리 사용량 확인
                    current_memory = process.memory_info().rss
                    if current_memory - initial_memory > self.memory_limit_bytes:
                        logger.warning(f"메모리 사용량 한계 접근: {(current_memory - initial_memory) / 1024 / 1024:.1f}MB")
                        
                        # 가비지 컬렉션 실행
                        import gc
                        gc.collect()
                    
                    # 청크 처리
                    processed_count = await self._process_chunk(
                        chunk_data, processor_func, **kwargs
                    )
                    total_processed += processed_count
                    
                    if total_chunks % 10 == 0:  # 10개 청크마다 로그
                        logger.info(f"처리 진행률: {total_chunks}개 청크, {total_processed}개 레코드 완료")
                
                except Exception as e:
                    failed_chunks += 1
                    error_info = {
                        "chunk_number": total_chunks,
                        "error": str(e),
                        "data_sample": str(chunk_data.head(3) if hasattr(chunk_data, 'head') else chunk_data)[:200]
                    }
                    processing_errors.append(error_info)
                    logger.error(f"청크 {total_chunks} 처리 실패: {e}")
                    
                    # 연속 실패 체크
                    if failed_chunks >= 5:
                        logger.error("연속 5개 청크 처리 실패, 작업 중단")
                        break
            
            # 최종 메모리 사용량
            final_memory = process.memory_info().rss
            memory_used = (final_memory - initial_memory) / 1024 / 1024
            
            return {
                "total_processed": total_processed,
                "total_chunks": total_chunks,
                "failed_chunks": failed_chunks,
                "success_rate": round((total_chunks - failed_chunks) / total_chunks * 100, 2) if total_chunks > 0 else 0,
                "memory_used_mb": round(memory_used, 2),
                "processing_errors": processing_errors[:10]  # 최대 10개 오류만 반환
            }
            
        except Exception as e:
            logger.error(f"대용량 데이터 처리 실패: {e}")
            raise
```

## 🚨 에러 처리 및 모니터링 규칙

### 1. 배치 에러 처리 패턴

```python
from enum import Enum
from typing import Dict, Any, Optional
import logging
import traceback
from datetime import datetime

class BatchErrorSeverity(Enum):
    """배치 오류 심각도"""
    LOW = "LOW"
    MEDIUM = "MEDIUM"
    HIGH = "HIGH"
    CRITICAL = "CRITICAL"

class BatchJobError(Exception):
    """배치 작업 오류"""
    def __init__(self, message: str, error_code: str = None, severity: BatchErrorSeverity = BatchErrorSeverity.MEDIUM):
        super().__init__(message)
        self.error_code = error_code
        self.severity = severity
        self.timestamp = datetime.now()

async def handle_batch_error(error: Exception, 
                           job_context: Dict[str, Any],
                           retry_strategy) -> Dict[str, Any]:
    """
    배치 오류 통합 처리
    
    Args:
        error: 발생한 오류
        job_context: 작업 컨텍스트 정보
        retry_strategy: 재시도 전략
    
    Returns:
        Dict[str, Any]: 오류 처리 결과
    """
    error_info = {
        "job_name": job_context.get("job_name"),
        "error_type": type(error).__name__,
        "error_message": str(error),
        "traceback": traceback.format_exc(),
        "timestamp": datetime.now().isoformat(),
        "job_context": job_context
    }
    
    # 오류 심각도 판단
    severity = _determine_error_severity(error, job_context)
    error_info["severity"] = severity.value
    
    # 재시도 가능 여부 확인
    attempt = job_context.get("retry_attempt", 0)
    if retry_strategy.should_retry(error, attempt):
        retry_delay = retry_strategy.get_retry_delay(attempt)
        error_info.update({
            "will_retry": True,
            "retry_attempt": attempt + 1,
            "retry_delay_seconds": retry_delay
        })
    else:
        error_info["will_retry"] = False
        # 오류 알림 발송
        await _send_error_notification(error_info)
    
    return error_info
```

## 🔧 개발 워크플로우

### 1. 배치 작업 개발 체크리스트

```bash
# 1. 코드 품질 검사
ruff check --fix .
black .
mypy app/

# 2. 배치 작업 테스트
pytest tests/unit/ -v
pytest tests/integration/ -v

# 3. API 키 관리 테스트
python -c "from app.core.multi_api_key_manager import MultiApiKeyManager; print('API 키 관리 시스템 정상')"

# 4. 데이터 처리 파이프라인 테스트
python -m app.processors.data_transformation_pipeline

# 5. 환경변수 검증
python debug_env.py
```

### 2. 배치 작업 개발 순서

1. **기본 작업 클래스 구현** (BaseJob 상속)
2. **데이터 수집/처리 로직** 구현
3. **에러 처리 및 재시도** 전략 설정
4. **스케줄 등록** (APScheduler 사용)
5. **테스트 작성** (단위/통합 테스트)
6. **모니터링 메트릭** 추가
7. **문서화** 업데이트

이 규칙들을 엄격히 준수하여 안정적이고 확장 가능한 Weather Flick 배치 시스템을 개발하세요.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/aicc6) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
