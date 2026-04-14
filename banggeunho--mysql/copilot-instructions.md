## mysql

> > 이 문서는 Claude가 동일한 MySQL 성능 최적화 학습 프로젝트를 재현할 수 있도록 하는 상세한 가이드입니다.

# MySQL 성능 최적화 마스터 과정 - 프로젝트 가이드

> 이 문서는 Claude가 동일한 MySQL 성능 최적화 학습 프로젝트를 재현할 수 있도록 하는 상세한 가이드입니다.

---

## 🚨 중요: Claude 작업 가이드라인

### 📝 Phase별 README 작성 원칙

각 Phase의 `docs/phase-XX/README.md` 작성 시 반드시 다음 내용을 포함해야 합니다:

#### 1. 개념 설명 (Why & What)
- **핵심 개념의 이론적 배경**: 단순히 "무엇을 하는가"가 아니라 "왜 이것이 필요한가"를 설명
- **MySQL 내부 동작 원리**: InnoDB 엔진 수준에서 어떻게 동작하는지 설명
- **성능에 미치는 영향**: 정량적인 수치와 함께 설명 (예: "인덱스 추가로 O(n) → O(log n)")

#### 2. 실무 예시 (Real-World Scenarios)
- **실제 서비스 사례**: "전자상거래에서 주문 조회 시...", "SNS 피드 조회 시..." 등 구체적 상황
- **장애 사례**: "인덱스 없이 운영하다가 발생한 장애 시나리오"
- **최적화 전/후 비교**: 실제 프로덕션에서 겪을 수 있는 Before/After 사례
- **안티패턴**: 실무에서 흔히 하는 실수와 그 결과

#### 3. 핵심 용어 정리
- 해당 Phase에서 다루는 모든 기술 용어에 대한 명확한 정의
- MySQL 공식 문서 링크 포함

#### 4. 실습 가이드
- 단계별 실습 순서
- 예상 결과값
- 트러블슈팅 가이드

**예시 (Phase 5 - 동시성 제어):**
```markdown
## 트랜잭션 격리 수준이란?

### 왜 필요한가?
동시에 여러 사용자가 같은 데이터를 읽고 쓸 때 발생하는 문제를 제어합니다.
예를 들어, 쇼핑몰에서 재고가 1개 남은 상품을 두 명이 동시에 구매하려 할 때...

### 실무 사례: 은행 송금 시스템
A가 B에게 100만원을 송금하는 도중 C가 A의 잔액을 조회하면?
- READ UNCOMMITTED: 송금 중간 상태(잔액 감소)가 보임 → Dirty Read
- READ COMMITTED: 송금 완료 전까지 이전 잔액이 보임
- REPEATABLE READ: 트랜잭션 시작 시점의 잔액이 계속 보임
- SERIALIZABLE: 송금이 완료될 때까지 조회 대기

### 안티패턴: SERIALIZABLE 남용
"안전하니까 모든 트랜잭션에 SERIALIZABLE 적용"
→ 결과: 동시성 처리량 90% 감소, 데드락 빈발
→ 해결: 대부분 READ COMMITTED로 충분, 필요한 곳만 락 명시적 사용
```

---

### 🔍 MySQL 8.0 호환성 검증 필수

**작업 완료 후 반드시 다음 검증을 수행해야 합니다:**

#### 1. 모든 SQL 스키마 실행 테스트
```bash
# 각 Phase의 SQL 파일을 실제로 실행하여 검증
docker exec mysql-performance-master mysql -u root -p[PASSWORD] < schemas/phase-XX/파일명.sql

# 에러 없이 실행되는지 확인
echo $?  # 0이면 성공
```

#### 2. 쉘 스크립트 실행 테스트
```bash
# 각 스크립트가 실제로 동작하는지 확인
bash scripts/phase-XX/스크립트명.sh
```

#### 3. 주요 검증 항목
- [ ] `information_schema.innodb_locks` → MySQL 8.0에서 `performance_schema.data_locks`로 변경됨
- [ ] `information_schema.innodb_lock_waits` → `performance_schema.data_lock_waits`로 변경됨
- [ ] Stored Procedure 파라미터에 `DEFAULT` 값 사용 불가
- [ ] 생성 컬럼(GENERATED)에서 `CURDATE()`, `NOW()` 등 비결정적 함수 사용 불가
- [ ] `LIMIT/OFFSET`에 표현식 직접 사용 불가 (Prepared Statement 필요)
- [ ] macOS에서 `date +%s.%N` 미지원 (python3 time.time() 사용)
- [ ] macOS에서 `paste -sd,` 동작 다름 (for 루프로 대체)

#### 4. 검증 자동화 명령어
```bash
# Phase 1-6 전체 스키마 검증
for phase in 01 02 03 04 05 06; do
  for sql in schemas/phase-${phase}/*.sql; do
    echo "Testing: $sql"
    docker exec -i mysql-performance-master mysql -u root -p[PASSWORD] < "$sql" 2>&1 | grep -i error
  done
done
```

#### 5. 검증 실패 시 조치
- 에러 메시지를 분석하여 MySQL 8.0 호환성 문제인지 확인
- 필요시 스키마/스크립트 수정
- 수정 후 반드시 재검증
- 모든 수정 사항을 커밋에 포함

---

## 📋 프로젝트 개요

### 사용자 요구사항
```
"docker로 mysql 띄우고, mysql로 대용량 데이터를 핸들링하는 법을 학습할거야"

추가 요구사항:
- 스텝별로 느린 것도 확인하면서 점차 개선하는 걸 보고 싶어
- 예를 들면 인덱스를 활용하기 전/후 비교
- 실행계획 학습, 옵티마이저 컨트롤
- 락이 걸리는 부분, 락을 해결하는 법
- 데드락이 걸리는 시연 해결하는 법
- 인덱스로만 해결되지 않는 부분은 어떻게 해결하는지
- 모니터링은 어떻게 하는지 등등 순차적으로 배우고 싶어
- 세세하게 학습 가이드와 스크립트를 작성해줘
- 백엔드 시니어 개발자로서 더 알아야 되는 부분 있으면 컨텐츠 추가
- mysql 설정에 관련한 컨텐츠도 있으면 좋을 것 같아(파라미터 라던가, 문자열 셋이라던가, 그리고 튜닝 가능한 부분 등등)
- 정규화에 대한 한계와 반정규화로 성능을 끌어올리는 예시도 있으면 좋을 것 같아
```

### 핵심 학습 목표
1. **체험형 학습**: 느린 상황부터 시작해서 단계적 성능 개선
2. **정량적 측정**: 모든 최적화의 Before/After 성능 비교
3. **실무 적용**: 백엔드 시니어 개발자 수준의 MySQL 운영 노하우
4. **포괄적 커버리지**: 설정→스키마→인덱싱→동시성→확장성→운영

## 🏗️ 프로젝트 구조

### 디렉토리 구조
```
mysql-performance-master/
├── README.md                      # 프로젝트 개요 및 전체 가이드
├── QUICK_START.md                 # 5분 빠른 시작 가이드
├── CLAUDE.md                      # 프로젝트 재현 가이드 (이 파일)
├── docker/                        # Docker 환경 설정
│   ├── docker-compose.yml        # MySQL + 모니터링 스택
├── config/                        # MySQL 설정 파일
│   ├── my.cnf                    # 마스터 서버 최적화 설정
│   └── my-slave.cnf              # 슬레이브 서버 설정
├── schemas/                       # 데이터베이스 스키마
│   ├── init/                     # 초기화 SQL
│   ├── normalized/               # 3NF 정규화 스키마
│   ├── denormalized/             # 반정규화 스키마
│   └── partitioned/              # 파티션 테이블 스키마
├── scripts/                       # 실행 스크립트
│   ├── phase-01/                 # Phase별 스크립트
│   ├── data-generation/          # 대용량 데이터 생성
│   ├── performance-testing/      # 성능 테스트 도구
│   └── monitoring/               # 모니터링 스크립트
├── examples/                      # 실습 예제
│   ├── charset-collation/        # 문자셋 성능 비교
│   ├── connection-tuning/        # 커넥션 풀 튜닝
│   ├── indexing/                 # 인덱싱 최적화
│   ├── normalization/            # 정규화 vs 반정규화
│   ├── concurrency/              # 동시성 제어
│   └── partitioning/             # 파티셔닝 전략
├── docs/                         # 단계별 학습 가이드
│   ├── phase-01/ ~ phase-11/     # 각 Phase별 상세 가이드
│   └── troubleshooting/          # 문제 해결 가이드
├── monitoring/                   # 모니터링 설정
│   ├── prometheus/               # Prometheus 설정
│   └── grafana/                  # Grafana 대시보드
└── performance-tests/            # 성능 테스트 결과
    ├── benchmarks/               # 벤치마크 결과
    └── reports/                  # 성능 분석 리포트
```

## 📚 11 Phase 학습 로드맵

### **Phase 1: 환경 구축 & MySQL 설정 마스터**
**목표**: 성능 최적화를 위한 완벽한 테스트 환경 구축

**주요 구성요소**:
- Docker Compose: MySQL 8.0 + phpMyAdmin + Redis + 모니터링
- MySQL 설정 최적화: InnoDB 파라미터, 메모리, 커넥션 풀
- 문자셋/콜레이션: utf8mb4 성능 영향 분석
- 슬로우 쿼리 로그 & Performance Schema 활성화

**핵심 파일**:
- `docker/docker-compose.yml`: 전체 스택 구성
- `config/my.cnf`: 성능 최적화된 MySQL 설정
- `examples/charset-collation/charset-performance-test.sql`: 문자셋 성능 비교
- `scripts/phase-01/test-environment.sh`: 환경 검증 스크립트

### **Phase 2: 데이터 모델링 & 정규화/반정규화**
**목표**: 정규화와 반정규화의 성능 차이 체험

**주요 학습**:
- 3NF 정규화 vs 반정규화 스키마 설계
- 복잡한 JOIN 쿼리의 성능 문제 체험
- 집계 테이블과 계산된 컬럼 활용
- 스토리지 vs 성능 트레이드오프 분석

**핵심 파일**:
- `schemas/normalized/01-create-tables.sql`: 완전 정규화 스키마
- `schemas/denormalized/01-create-tables.sql`: 성능 최적화 반정규화 스키마
- 대용량 데이터 생성기 (100만~1000만 건)

### **Phase 3: "느린 상황" 체험 & 기본 최적화**
**목표**: 인덱스 없는 상황부터 시작해 점진적 성능 개선

**체험 시나리오**:
1. 인덱스 없는 테이블에서 Full Table Scan 체험
2. EXPLAIN으로 실행계획 분석
3. 슬로우 쿼리 로그 분석
4. 기본 인덱스 추가 후 성능 향상 측정

### **Phase 4: 인덱싱 전략 마스터**
**목표**: 다양한 인덱싱 기법과 성능 최적화

**주요 기법**:
- 단일 vs 복합 인덱스 성능 비교
- 커버링 인덱스로 JOIN 제거
- 인덱스 힌트와 옵티마이저 제어
- JSON 컬럼 & 풀텍스트 인덱스 활용

### **Phase 5: 동시성 제어 & 락 관리** ✅ 완료
**목표**: 락 문제 체험과 해결 방법 학습

**핵심 파일**:
- `schemas/phase-05/01-transaction-isolation-locks.sql`: 격리 수준별 락 동작
- `schemas/phase-05/02-deadlock-demonstration-resolution.sql`: 데드락 시연 및 해결
- `schemas/phase-05/03-gap-lock-next-key-analysis.sql`: Gap Lock 분석
- `schemas/phase-05/04-application-level-concurrency.sql`: 앱 레벨 동시성 제어

**실습 시나리오**:
- 트랜잭션 격리 수준별 락 동작 체험 (READ UNCOMMITTED ~ SERIALIZABLE)
- 데드락 의도적 발생 및 자동 복구 메커니즘 분석
- Gap Lock & Next-Key Lock 심화 분석 및 시각화
- 낙관적/비관적 락, 세마포어 패턴 구현

### **Phase 6: 고급 최적화 & 파티셔닝** ✅ 완료
**목표**: 인덱스로 해결되지 않는 문제의 고급 해결책

**핵심 파일**:
- `schemas/phase-06/01-partitioning-strategies.sql`: 파티셔닝 전략
- `schemas/phase-06/02-batch-processing.sql`: 배치 처리 최적화
- `schemas/phase-06/03-temp-memory-tables.sql`: 임시/메모리 테이블
- `schemas/phase-06/04-procedure-vs-application.sql`: 프로시저 vs 앱 로직

**고급 기법**:
- RANGE/LIST/HASH/KEY 파티셔닝 성능 비교
- 배치 처리 최적화 (LOAD DATA, bulk insert, 청크 처리)
- 임시 테이블 vs 메모리 테이블 사용 전략
- 프로시저 vs 애플리케이션 로직 성능 비교

### **Phase 7: (예정)**
**목표**: 추가 고급 최적화 기법

### **Phase 8: 확장성 & 아키텍처 패턴**
**목표**: 대용량 트래픽 처리를 위한 확장 전략

**확장 기법**:
- 마스터-슬레이브 복제 구성 및 지연 모니터링
- 샤딩 전략 설계 (범위 기반, 해시 기반)
- 읽기 전용 복제본 활용 패턴
- 캐시 계층 연동 (Redis) 및 무효화

### **Phase 9: 운영 & 백업/복원**
**목표**: 실무 운영 필수 기술

**운영 기술**:
- 논리적/물리적 백업 전략 및 복원 테스트
- 바이너리 로그 관리 및 포인트-인-타임 복구
- 테이블 스페이스 관리 및 디스크 I/O 최적화
- 로그 로테이션 및 스토리지 관리

### **Phase 10: 모니터링 & 알림 시스템**
**목표**: 실시간 성능 모니터링과 장애 대응

**모니터링 시스템**:
- Performance Schema 활용한 실시간 모니터링
- Grafana 대시보드 구축
- 성능 지표 알림 및 임계값 설정
- 장애 대응 플레이북 및 자동화

### **Phase 11: 실전 시나리오 & 종합 테스트**
**목표**: 실제 서비스 수준의 성능 테스트

**종합 시나리오**:
- 전자상거래 실시간 주문 처리 시뮬레이션
- 대용량 트래픽 부하 테스트 (동시 접속 1000+)
- 실제 장애 시나리오 재현 및 복구 실습
- 성능 최적화 종합 보고서

## 🎯 데이터 규모 설정

**학습 단계별 데이터 볼륨**:
- **기본 학습**: 100만 건 (빠른 테스트, 개념 이해)
- **실전 체험**: 500만 건 (실무 수준 성능 체험)
- **심화 학습**: 1,000만 건 (대용량 처리 최적화)

**주요 테이블 구성**:
- users: 사용자 정보 (100만 건)
- products: 제품 정보 (10만 건)
- orders: 주문 정보 (500만 건)
- order_items: 주문 상품 (1,500만 건)
- activity_logs: 시스템 로그 (1억 건 - 파티션 테이블)

## 🔧 핵심 MySQL 설정

### InnoDB 최적화 설정
```ini
# 메모리 설정
innodb_buffer_pool_size = 1G           # 전체 RAM의 70-80%
innodb_log_buffer_size = 64M
innodb_log_file_size = 256M

# I/O 최적화
innodb_flush_log_at_trx_commit = 2     # 성능 vs 안정성 균형
innodb_flush_method = O_DIRECT
innodb_file_per_table = 1

# 동시성 설정
innodb_thread_concurrency = 0          # 자동 조정
innodb_read_io_threads = 8
innodb_write_io_threads = 8
```

### 성능 모니터링 설정
```ini
# 슬로우 쿼리 로그
slow_query_log = 1
long_query_time = 0.1
log_queries_not_using_indexes = 1

# Performance Schema
performance_schema = 1
performance-schema-instrument = 'statement/%=ON'
```

## 📊 Before/After 성능 측정 방법론

### 1. 기준선 측정
```sql
-- 실행 시간 측정
SET @start_time = NOW(6);
[테스트 쿼리]
SET @end_time = NOW(6);
SELECT TIMESTAMPDIFF(MICROSECOND, @start_time, @end_time) / 1000 as execution_time_ms;

-- 리소스 사용량 측정
SELECT * FROM performance_schema.memory_summary_global_by_event_name;
```

### 2. 실행계획 분석
```sql
-- 상세 실행계획
EXPLAIN FORMAT=JSON [쿼리];

-- 비용 분석
SELECT * FROM performance_schema.events_statements_history_long
WHERE SQL_TEXT LIKE '%your_query%';
```

### 3. 성능 개선 후 재측정
```sql
-- 동일한 조건에서 재측정
-- 개선율 계산: ((old_time - new_time) / old_time) * 100
```

## 🚀 프로젝트 재현 가이드

### 1. 환경 구축 (Phase 1)
```bash
# 1. 프로젝트 디렉토리 생성
mkdir mysql-performance-master && cd mysql-performance-master

# 2. 필수 디렉토리 구조 생성
mkdir -p {docker,config,schemas/{init,normalized,denormalized,partitioned},scripts/{phase-01,data-generation,performance-testing,monitoring},examples/{charset-collation,connection-tuning,indexing,normalization,concurrency,partitioning},docs/{phase-01,phase-02,phase-03,phase-04,phase-05,phase-06,phase-07,phase-08,phase-09,phase-10,phase-11},monitoring/{prometheus,grafana/provisioning},performance-tests/{benchmarks,reports}}

# 3. 핵심 파일 작성 (이 가이드 참조)
# - docker/docker-compose.yml
# - config/my.cnf
# - schemas/init/01-create-databases.sql
# - scripts/phase-01/test-environment.sh
# - README.md, QUICK_START.md

# 4. 환경 시작
cd docker && docker-compose up -d
```

### 2. 학습 진행 방식
1. **각 Phase별 README 먼저 읽기**
2. **환경 테스트 스크립트 실행**
3. **Before 성능 측정**
4. **최적화 적용**
5. **After 성능 측정 및 비교**
6. **다음 Phase로 진행**

### 3. 품질 검증
- 모든 최적화는 정량적 성능 측정으로 검증
- Before/After 비교 결과를 문서화
- 실무 적용 가능한 베스트 프랙티스 도출

## 🎓 학습 결과물

### 각 Phase 완료 시 산출물
- **성능 측정 리포트**: Before/After 비교 분석
- **최적화 과정 문서**: 단계별 적용 과정
- **실무 체크리스트**: 바로 적용 가능한 가이드
- **장애 대응 플레이북**: 문제 상황별 해결 방법

### 최종 습득 역량
- MySQL 성능 튜닝 전 영역 마스터
- 대용량 데이터 처리 아키텍처 설계 능력
- 실시간 모니터링 및 장애 대응 역량
- 백엔드 시니어 개발자 수준의 MySQL 운영 노하우

## 💡 프로젝트 확장 가능성

### 추가 학습 모듈
- **MySQL 8.0 신기능**: 윈도우 함수, CTE, JSON 최적화
- **클라우드 환경**: AWS RDS, Aurora 성능 최적화
- **마이크로서비스**: DB 분리 및 데이터 일관성
- **NoSQL 비교**: MongoDB, Redis 성능 비교 분석

### 실무 확장
- **CI/CD 통합**: 성능 테스트 자동화
- **모니터링 확장**: Prometheus + Grafana 고도화
- **장애 복구**: 자동 failover 및 복구 시스템
- **보안 강화**: 암호화, 접근 제어, 감사 로그

이 가이드를 따라 구현하면 동일한 수준의 MySQL 성능 최적화 학습 환경을 구축할 수 있습니다. 각 Phase는 독립적으로도 활용 가능하며, 전체 과정을 통해 체계적인 MySQL 마스터가 될 수 있습니다.

---

## ✅ Phase 완료 체크리스트

각 Phase 작업 완료 시 아래 항목을 모두 확인해야 합니다:

### 1. 코드 품질
- [ ] 모든 SQL 파일이 MySQL 8.0에서 에러 없이 실행됨
- [ ] 모든 쉘 스크립트가 macOS/Linux에서 정상 동작
- [ ] DROP TABLE IF EXISTS로 멱등성 보장
- [ ] 외래키 제약조건 순서 고려 (SET FOREIGN_KEY_CHECKS = 0)

### 2. 문서 품질 (docs/phase-XX/README.md)
- [ ] **개념 설명**: 해당 기술의 이론적 배경과 MySQL 내부 동작 원리
- [ ] **실무 예시**: 최소 3개 이상의 실제 서비스 사례
- [ ] **안티패턴**: 실무에서 흔히 하는 실수와 그 결과
- [ ] **성능 수치**: Before/After 정량적 비교 데이터
- [ ] **용어 정리**: 핵심 기술 용어 정의 및 MySQL 공식 문서 링크
- [ ] **실습 가이드**: 단계별 실습 순서와 예상 결과값
- [ ] **트러블슈팅**: 자주 발생하는 문제와 해결 방법

### 3. 검증 완료
- [ ] Docker 컨테이너에서 모든 스키마 실행 테스트 완료
- [ ] 모든 스크립트 실행 테스트 완료
- [ ] 성능 테스트 쿼리 실행 및 결과 확인
- [ ] 에러 발생 시 수정 및 재검증 완료

### 4. 커밋
- [ ] 의미 있는 커밋 메시지 작성
- [ ] 변경 내용 요약 포함
- [ ] MySQL 8.0 호환성 수정 사항 명시

---

## 📋 Phase별 README 템플릿

```markdown
# Phase X: [제목]

## 🎯 학습 목표
[이 Phase에서 달성하고자 하는 구체적인 목표]

## 📚 핵심 개념

### [개념 1]: [제목]

#### 왜 필요한가?
[비즈니스/기술적 필요성 설명]

#### MySQL 내부 동작 원리
[InnoDB 엔진 수준의 동작 설명]

#### 실무 사례
**사례 1: [서비스명] - [상황]**
- 문제: [구체적인 문제 상황]
- 원인: [기술적 원인 분석]
- 해결: [적용한 해결책]
- 결과: [정량적 개선 수치]

**사례 2: ...**

#### 안티패턴
❌ **[잘못된 패턴]**
- 증상: [어떤 문제가 발생하는가]
- 원인: [왜 문제가 되는가]
- 해결: [올바른 방법]

### [개념 2]: ...

## 🔧 실습 가이드

### Step 1: [제목]
```sql
[실행할 SQL]
```
**예상 결과:**
[예상되는 출력 또는 결과]

### Step 2: ...

## 📊 성능 비교

| 시나리오 | Before | After | 개선율 |
|---------|--------|-------|--------|
| [시나리오1] | Xms | Yms | Z% |

## 🔗 참고 자료
- [MySQL 공식 문서 - 해당 주제](링크)
- [추가 참고 자료](링크)

## ❓ FAQ / 트러블슈팅

### Q: [자주 묻는 질문]
A: [답변]

### 문제: [에러 메시지]
해결: [해결 방법]
```

---

## 🔄 작업 흐름 요약

```
1. Phase 스키마/스크립트 작성
   ↓
2. Docker에서 실행 테스트
   ↓
3. 에러 발생 시 수정 및 재테스트
   ↓
4. docs/phase-XX/README.md 작성 (개념 + 실무 예시 + 안티패턴)
   ↓
5. 전체 검증 (스키마 + 스크립트 + 문서)
   ↓
6. 커밋 (상세한 변경 내용 포함)
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/banggeunho)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/banggeunho)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
