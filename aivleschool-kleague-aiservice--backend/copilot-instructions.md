## backend

> 이 파일은 `backend/` 전체에 적용된다.

# K리그 백엔드 리팩토링 작업 지침

## 적용 범위

이 파일은 `backend/` 전체에 적용된다.

## 프로젝트 목표

K리그 승부예측 서비스의 실제 사용자 조회 경로와 데이터 적재 경로를 측정하고 개선하여, 재현 가능한 포트폴리오 근거를 만든다.

다음 두 문제를 독립적으로 다룬다.

1. 경기 상세 조회 성능: API 기준선, 통합 API, 조건부 Redis
2. CSV 적재 성능: Row-by-Row 기준선, JDBC Batch

## 실행 책임

- 사용자가 IntelliJ IDEA의 Run 모드로 Spring Boot를 실행한다.
- 사용자가 k6 워밍업과 기록용 성능 테스트를 직접 실행한다.
- 에이전트는 사용자의 요청 없이 Spring Boot 프로세스를 실행하거나 종료하지 않는다.
- 에이전트는 k6 스크립트와 실행 명령을 준비하고, 사용자가 전달한 결과를 분석·문서화한다.
- 필요한 경우 `http://localhost:8080`의 준비 상태만 확인한다.
- 성능 측정 중 IntelliJ Debug 모드를 사용하지 않는다.
- MySQL·Redis의 데이터 삭제나 초기화가 필요하면 정확한 대상과 복구 방법을 먼저 확인한다.

## 기준 문서

작업은 다음 문서를 순서대로 진행한다.

| 단계 | 문서 | 목적 | 상태 |
|---:|---|---|---|
| 1 | `docs/refactoring/01_API_BASELINE.md` | 현재 조회 구조의 Before 측정 | Completed |
| 2 | `docs/refactoring/02_ROUND_QUERY_N_PLUS_ONE.md` | 라운드 목록 N+1 판단 | Completed |
| 3 | `docs/refactoring/03_UNIFIED_TIMELINE_API.md` | API 3개를 통합 API로 전환 | Completed |
| 4 | `docs/refactoring/04_REDIS_CACHE.md` | Redis 학습과 측정 근거 기반 조건부 캐싱 | Completed |
| 5 | `docs/refactoring/05_CSV_IMPORT_BASELINE.md` | Row-by-Row 적재 기준선 | In Progress |
| 6 | `docs/refactoring/06_JDBC_BATCH_INSERT.md` | JDBC Batch 적재 개선 | Not Started |
| 7 | `docs/refactoring/07_PORTFOLIO_AND_BLOG.md` | 결과 기반 포트폴리오·블로그 작성 | Not Started |

## 현재 단계

- Current: `05_CSV_IMPORT_BASELINE.md`
- Status: In Progress
- Next: `06_JDBC_BATCH_INSERT.md`

한 번에 하나의 단계만 `In Progress`로 둔다.

## 전역 규칙

1. 개선 전에 반드시 Before 수치를 측정한다.
2. 현재 단계 이후의 기능을 미리 구현하지 않는다.
3. 통합 타임라인 API와 Redis를 동시에 적용하지 않는다.
4. 성능 측정 중 `show-sql`과 상세 SQL 콘솔 출력을 끈다.
5. SQL 수 확인과 순수 응답 시간 측정을 분리한다.
6. 전후 비교는 같은 장비, JVM, 데이터, DB 설정, 테스트 스크립트를 사용한다.
7. 각 시나리오는 워밍업 후 최소 3회 반복한다.
8. 평균만 사용하지 않고 p50·p95·p99·RPS·오류율을 기록한다.
9. 데이터 행 수와 값이 변경되면 성능 개선으로 인정하지 않는다.
10. 로컬 측정값을 운영 서버의 처리 한계처럼 표현하지 않는다.
11. 가설과 결과가 다르면 결과에 맞춰 결론을 수정한다.
12. Redis, 인덱스, 비동기 기술을 학습 목적만으로 성능 해결책처럼 포장하지 않는다.
13. Redis 전후 공식 비교는 같은 현재 빌드에서 Cache Disabled와 Warm Cache로 측정한다.
14. Cold Start는 첫 캐시 생성 비용과 동시 Miss를, Warm Cache는 지속 부하 성능을 검증하며 두 결과를 섞지 않는다.
15. 04단계 결과가 확정되면 Redis 포트폴리오 초안을 먼저 남긴 뒤 독립적인 CSV 적재 트랙으로 이동한다.

## 작업 시작 절차

1. 이 `AGENTS.md`를 읽는다.
2. 현재 단계 문서만 읽는다.
3. 선행 단계의 Evidence가 필요한 경우 해당 결과만 확인한다.
4. 현재 Git 상태와 사용자 변경을 확인한다.
5. 현재 단계의 금지 사항을 확인한다.
6. 작업 범위와 측정 조건을 짧게 보고한 뒤 진행한다.

## 단계 완료 절차

각 단계가 끝나면 다음 내용을 보고한다.

- 변경 파일
- 실행 명령
- 테스트 결과
- 측정 환경
- 측정 수치
- 원본 데이터와 결과 데이터의 동등성
- 발견한 문제
- 가설과 실제 결과의 차이
- 생성한 Evidence 파일
- 다음 단계 진입 가능 여부

완료 조건을 충족하지 못하면 해당 단계를 `Completed`로 표시하지 않는다.

## Evidence 규칙

- 계획은 `docs/refactoring/`에 둔다.
- 결과는 `docs/evidence/<단계>/`에 둔다.
- 재실행 가능한 k6 스크립트는 `performance/k6/`에 둔다.
- 일회성 대용량 로그, DB 덤프, 비밀번호는 커밋하지 않는다.
- 포트폴리오의 모든 수치는 Evidence와 재실행 방법으로 추적할 수 있어야 한다.

## 검증 기준

코드 변경 단계에서는 최소한 다음을 실행한다.

```bash
./gradlew test
./gradlew compileJava
```

성능 테스트는 기능 테스트를 통과한 뒤 수행한다.

## Git 안전 규칙

- 기존 사용자 변경과 `.DS_Store` 변경을 임의로 되돌리거나 커밋하지 않는다.
- 요청받지 않은 파일을 포맷하거나 정리하지 않는다.
- 사용자의 명시적 요청 없이 commit, push, PR 생성을 하지 않는다.
- 단계별 변경은 다른 단계의 작업과 섞지 않는다.

---
> Source: [Aivleschool-Kleague-AIService/backend](https://github.com/Aivleschool-Kleague-AIService/backend) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-26 -->
