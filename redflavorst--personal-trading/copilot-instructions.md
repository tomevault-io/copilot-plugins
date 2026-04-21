## personal-trading

> - 키움증권 API를 활용한 트레이딩 시스템

# 키움 프로젝트 지침

## 프로젝트 개요
- 키움증권 API를 활용한 트레이딩 시스템
- Python 기반 프로젝트

## 코딩 스타일
- Python 파일은 UTF-8 인코딩 사용
- 들여쓰기: 스페이스 4칸
- 함수/변수명: snake_case 사용 (예: `get_daily_data`, `trade_grouper`)
- 클래스명: PascalCase 사용 (예: `TradeGrouper`, `DBManager`)
- 한글 주석 허용 (필요시 한글로 설명)

## 파일 구조
- `src/api/`: API 관련 코드 (키움 API 연동)
- `src/ui/`: UI 관련 코드 (대시보드, 저널 등)
- `src/logic/`: 비즈니스 로직
- `src/data/`: 데이터 처리
- `src/database/`: DB 관련 코드
- `src/utils/`: 유틸리티 함수
- `scripts/`: 실행 스크립트

## 중요 규칙
- CSV 파일 처리 시 헤더 라인에 'v'가 있으면 제거
- 에러 처리는 반드시 명시적으로 작성 (try-except 사용)
- API 키나 민감한 정보는 절대 하드코딩하지 않고 config 파일 사용
- 데이터베이스 연결은 반드시 close 처리

## 테스트 및 실행
- 테스트 전에 키움 API 연결 상태 확인
- 메인 실행: `python src/main.py`

## 기타
- 주석은 한글/영어 혼용 가능
- 로그는 명확하게 남기기 (디버깅 용이하도록)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/redflavorst) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
