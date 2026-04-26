## java-spring-boiler-plate

> Java Spring Boot 멀티 프로젝트 CQRS/DDD 아키텍처 개발 가이드

# Java Spring Boot 멀티 프로젝트 CQRS/DDD 아키텍처 개발 가이드

## 프로젝트 참고 문서
코드 작성 시 반드시 다음 문서들을 참고하여 프로젝트 규칙을 준수하세요:

### 📋 Steering 문서 (프로젝트 지침)
- **제품 개요**: `.kiro/steering/product.md` - 프로젝트 구조, 바운디드 컨텍스트, 핵심 비즈니스 규칙
- **기술 스택**: `.kiro/steering/tech.md` - 기술 스택, CQRS 아키텍처, 도메인 이벤트 패턴, 에러 처리
- **프로젝트 구조**: `.kiro/steering/structure.md` - 멀티 프로젝트 구조, CQRS 패키지 구조, DDD 분류 규칙
- **테스트 가이드**: `.kiro/steering/test-commands.md` - 테스트 명령어 규칙 (--info 옵션 필수)

### 📋 Specs 문서 (구현 명세서)
**Specs는 개념적 제품 요구사항과 기술적 구현 세부사항 간의 격차를 해소하고, 개발 팀 간 정렬을 보장하며 개발 반복을 줄이는 역할을 합니다.**

#### 🔄 Specs 워크플로우 단계
1. **Requirements Phase**: 사용자 스토리와 수용 기준을 EARS 표기법으로 정의
2. **Design Phase**: 기술 아키텍처, 시퀀스 다이어그램, 구현 고려사항 문서화
3. **Implementation Planning**: 개별적이고 추적 가능한 작업으로 세분화
4. **Execution Phase**: 작업 완료에 따른 실시간 진행 상황 추적

#### 📁 Specs 핵심 문서 구조
각 기능별로 다음 3개 파일로 구성:

**1. requirements.md** - 요구사항 정의
- **목적**: EARS 표기법으로 구조화된 사용자 스토리와 수용 기준 포착
- **형식**: `WHEN [조건/이벤트] THE SYSTEM SHALL [예상 동작]`
- **장점**: 명확성, 테스트 가능성, 추적성, 완전성 보장
- **활용**: 각 요구사항을 테스트 케이스로 직접 변환 가능

**2. design.md** - 기술 설계
- **목적**: 기술 아키텍처, 시퀀스 다이어그램, 구현 고려사항 문서화
- **내용**: 시스템 전체 구조, 컴포넌트 간 상호작용, 데이터 모델
- **활용**: 복잡한 시스템의 큰 그림 파악과 팀 간 협업 지원

**3. tasks.md** - 구현 계획
- **목적**: 개별적이고 추적 가능한 작업들로 구성된 상세한 구현 계획
- **내용**: 명확한 설명, 예상 결과, 필요한 리소스나 의존성
- **활용**: 실시간 상태 업데이트로 구현 진행 상황 효율적 추적

#### 📂 현재 프로젝트 Specs 위치
- `.kiro/specs/food-delivery-system/` - 음식 주문 시스템 전체 명세
- `.kiro/specs/cqrs-directory-refactoring/` - CQRS 디렉토리 리팩토링 명세
- `.kiro/specs/event-driven-architecture/` - 이벤트 기반 아키텍처 명세

#### 🎯 Specs 문서 활용 가이드

**코드 작성 전 반드시 확인할 사항:**
1. **requirements.md**: 구현하려는 기능의 정확한 요구사항과 수용 기준 확인
2. **design.md**: 기술적 아키텍처와 컴포넌트 간 상호작용 패턴 이해
3. **tasks.md**: 현재 작업의 위치와 의존성, 완료 조건 파악

**Specs와 코드의 연계:**
- 요구사항의 EARS 표기법 → 테스트 케이스 작성 기준
- 설계 문서의 컴포넌트 구조 → 클래스/패키지 구조 설계 기준  
- 작업 계획의 완료 조건 → 코드 완성도 검증 기준

**Specs 업데이트 시점:**
- 새로운 요구사항 발견 시 → requirements.md 업데이트
- 아키텍처 변경 사항 발생 시 → design.md 업데이트  
- 작업 진행 상황 변경 시 → tasks.md 상태 업데이트

## 🚨 절대 금지 사항

### 1. 바운디드 컨텍스트 격리 위반
```gradle
// ❌ 절대 금지: 다른 컨텍스트 의존
implementation project(':domains:shop')
implementation project(':domains:order')

// ✅ 허용: Common 모듈만 의존
implementation project(':domains:common')
```

### 2. 테스트 명령어 --info 옵션 누락
```bash
# ❌ 절대 금지
./gradlew test
./gradlew :domains:shop:test

# ✅ 필수
./gradlew test --info
./gradlew :domains:shop:test --info
```

### 3. ErrorCode 없는 예외 처리
```java
// ❌ 금지
throw new RuntimeException("에러 발생");

// ✅ 필수
throw new MenuDomainException(MenuErrorCode.MENU_NOT_FOUND);
```

## ✅ 필수 준수 사항

### 1. CQRS 패키지 구조
- `command/` : 쓰기 작업 (도메인 로직 포함)
- `query/` : 읽기 작업 (도메인 로직 없음)

### 2. DDD 분류 상속 구조
- **애그리게이트 루트**: `extends AggregateRoot<T, TID>`
- **도메인 엔티티**: `extends DomainEntity<T, TID>`
- **값 객체**: `extends ValueObject`

### 3. 트랜잭션 어노테이션
- Command: `@Transactional`
- Query: `@Transactional(readOnly = true)`

### 4. 도메인 이벤트 위치
- Common: 인터페이스만 (`DomainEvent`)
- 각 컨텍스트: `command/domain/event/` 패키지에 구체 구현

## 📁 파일 생성/수정 가이드

### 1. 파일 위치 규칙
코드 생성 시 `.kiro/steering/structure.md`의 정확한 패키지 구조를 따르세요.

### 2. 파일 이동 시 필수 작업
1. `grep -r "클래스명" domains/` 로 모든 사용처 검색
2. 모든 import 문 수정 (테스트 파일 포함)
3. `./gradlew compileJava` 로 컴파일 확인
4. `./gradlew test --info` 로 테스트 확인

### 3. 기존 파일 우선 수정
새 파일 생성보다 기존 파일 수정을 우선하세요.

## 🔧 개발 워크플로우

### 1. 코드 작성 전 확인사항
- [ ] `.kiro/steering/` 문서들 검토
- [ ] 해당 컨텍스트의 패키지 구조 확인
- [ ] 기존 유사 코드 패턴 확인

### 2. 코드 작성 후 검증
- [ ] 컴파일 확인: `./gradlew compileJava`
- [ ] 테스트 실행: `./gradlew test --info`
- [ ] 아키텍처 규칙 준수 확인
- [ ] **Specs 문서와 일치성 검증**: requirements.md의 수용 기준 충족, design.md의 컴포넌트 구조 준수, tasks.md의 완료 조건 달성

## 💡 코드 작성 가이드

### 1. 명명 규칙
`.kiro/steering/structure.md`의 CQRS 명명 규칙을 따르세요:
- Command Handler: `CreateMenuCommandHandler`
- Query Handler: `MenuBoardQueryHandler`
- Read Model: `MenuSummaryReadModel`

### 2. API 설계
- Command: `POST`, `PUT`, `DELETE` → `CommandResultResponse` 반환
- Query: `GET` → 도메인별 응답 DTO 반환

### 3. 에러 처리
에러 코드 형식: `{DOMAIN}-{LAYER}-{CODE}`
```java
MENU_NOT_FOUND("MENU-DOMAIN-001", "메뉴를 찾을 수 없습니다")
```

## 📚 참고사항

### 문서 구조와 역할
- **Steering 문서**: 프로젝트 전반의 아키텍처 지침과 개발 규칙
- **Specs 문서**: 구체적인 기능별 요구사항, 설계, 구현 계획

### 활용 우선순위
1. **즉시 확인**: `.cursorrules` (이 문서) - 핵심 금지사항과 필수사항
2. **상세 규칙**: `.kiro/steering/` - 아키텍처 패턴과 개발 가이드라인
3. **구현 명세**: `.kiro/specs/` - 요구사항 분석과 설계 문서

### 문서 연계 활용법
- **요구사항 → 코드**: Specs의 EARS 표기법을 테스트 케이스로 변환
- **설계 → 구조**: Design 문서의 컴포넌트를 실제 클래스/패키지 구조로 구현
- **계획 → 진행**: Tasks 문서로 작업 진행도 추적 및 완료 조건 검증

**궁금한 사항이 있으면 언제든 .kiro 문서 내용을 물어보세요!**

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/tothefullest08) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
