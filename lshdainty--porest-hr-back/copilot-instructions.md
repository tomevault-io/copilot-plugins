## porest-hr-back

> - Language: Java 17 [LTS 사용]

# Project Overview
- Language: Java 17 [LTS 사용]
- Framework: Spring Boot 3.4.x (Spring Framework 6.x)
- Build: Gradle
- Packaging: Jar 배포
- API 문서: springdoc-openapi (Swagger UI)

## SSO 연동 아키텍처

### 역할 분리
| 구분 | SSO | HR |
|------|-----|-----|
| 인증 정보 | ID, Password, Email | X (SSO 참조) |
| 초대/회원가입 | 초대 토큰, 상태, 만료일시 | X (SSO에서 관리) |
| 비즈니스 정보 | X | 회사, 근무시간, 입사일, 부서, 휴가 등 |

### HR User 엔티티
- `ssoUserRowId`: SSO와 HR 간 사용자 연결 키 (FK 역할)
- 인증 관련 필드 없음 (password, invitationStatus 등은 SSO에서 관리)
- 비즈니스 데이터만 관리: company, workTime, joinDate, birth, department, vacation 등

### SSO API 연동
```java
// SSO API 클라이언트 (client/sso/)
@Component
public class SsoApiClientImpl implements SsoApiClient {
    // 사용자 초대: POST /api/v1/users/invite
    SsoInviteResponse inviteUser(SsoInviteRequest request);

    // 초대 재발송: POST /api/v1/users/{userId}/invitations/resend
    void resendInvitation(String userId);
}
```

### 초대 프로세스
1. HR 관리자가 사용자 초대 → HR Backend가 SSO API 호출
2. SSO에서 User 생성 + UserClientAccess 생성 + 초대 이메일 발송
3. SSO 응답으로 받은 `ssoUserRowId`를 HR User에 저장
4. 사용자가 SSO에서 회원가입 완료 → Redis 이벤트로 HR에 알림

## Modules & Structure
- modules: core(domain), api(web), infra(persistence, external-clients) [web:27]
- Hexagonal 스타일: domain은 framework 의존 금지, adapter는 의존 허용 [web:26]
- 패키지 규칙: com.porest.hr.{domain}.{controller, service, repository} [web:26]

## Coding Standards
- Controller: DTO 변환 철저, 엔티티 노출 금지 [web:26]
- Service: 트랜잭션 경계 @Transactional, 읽기 전용 readOnly=true [web:26]
- Repository: QueryDSL [web:26]
  - Repository는 순수 데이터 접근만 담당 (비즈니스 로직 금지)
  - 날짜 변환, 값 계산 등은 Service에서 처리 후 파라미터로 전달
  - 재사용성과 테스트 용이성을 위해 계산된 값을 받도록 설계
  - **Repository 구조 (3개 파일 필수)**:
    - `{Entity}Repository.java` - 인터페이스 (JavaDoc 주석 포함)
    - `{Entity}QueryDslRepository.java` - QueryDSL 구현체 (`@Primary` 적용)
    - `{Entity}JpaRepository.java` - JPQL 구현체 (백업용)
  - **QueryDslRepository 수정 시 JpaRepository도 동기화 필수**
  - QueryDslRepository: `@Repository`, `@Primary`, `@RequiredArgsConstructor`
  - JpaRepository: `@Repository("{entity}JpaRepository")`, `@RequiredArgsConstructor`
- 예외: 전역 @RestControllerAdvice로 에러 응답 표준화 (상세 내용은 아래 "Exception 처리" 섹션 참조)
- 로깅: slf4j 사용 [web:26]
- Import 규칙:
  - 클래스는 반드시 파일 상단에 import 문으로 선언 (풀경로 인라인 사용 금지)
  - 사용하지 않는 import는 즉시 삭제
  - 와일드카드 import(`*`) 사용 금지, 개별 클래스 명시

## i18n & Message 관리
- 메시지 키: `MessageKey` enum으로 중앙 관리 (`common/message/MessageKey.java`)
- 에러 코드: `ErrorCode` enum으로 중앙 관리 (`common/exception/ErrorCode.java`)
- 메시지 조회: `MessageResolver` 사용 (`common/util/MessageResolver.java`)
- 메시지 파일 위치: `src/main/resources/message/`
  - `messages.properties` (기본, 영어)
  - `messages_en.properties` (영어)
  - `messages_ko.properties` (한국어)

### 메시지 키 네이밍 규칙
- **모두 소문자 + 점(.) 구분자 사용** (camelCase 금지)
- 올바른 예: `error.notfound.vacation.grant`
- 잘못된 예: `error.notFound.vacationGrant`

### 새 메시지 추가 절차 (필수)
**모든 메시지는 반드시 3개 파일 동기화 필요:**

1. `messages.properties`에 메시지 추가 (기본, 영어)
2. `messages_en.properties`에 영어 메시지 추가
3. `messages_ko.properties`에 한국어 메시지 추가
4. `MessageKey` 또는 `ErrorCode`에서 해당 키 참조

### ErrorCode 사용 (예외 처리용)
```java
// 1. ErrorCode에 정의
USER_NOT_FOUND("USER_001", "error.notfound.user", HttpStatus.NOT_FOUND),

// 2. messages.properties (3개 파일 모두)
error.notfound.user=User not found

// 3. 서비스에서 사용
throw new EntityNotFoundException(ErrorCode.USER_NOT_FOUND);
throw new InvalidValueException(ErrorCode.VACATION_INVALID_DATE);
throw new DuplicateException(ErrorCode.USER_ALREADY_EXISTS);
```

### MessageKey 사용 (일반 메시지 조회용)
```java
// 1. MessageKey에 정의
NOT_FOUND_USER("error.notfound.user"),

// 2. 서비스에서 사용
String message = messageResolver.getMessage(MessageKey.NOT_FOUND_USER);
// 파라미터 포함 시
String message = messageResolver.getMessage(MessageKey.FILE_READ, fileName);
```

### 카테고리별 접두어
| 카테고리 | 접두어 | 예시 |
|---------|--------|------|
| 조회 실패 | `error.notfound.*` | `error.notfound.user` |
| 검증 오류 | `error.validate.*` | `error.validate.parameter.null` |
| 도메인별 | `error.{domain}.*` | `error.vacation.invalid.date` |
| 휴가 정책 | `vacation.policy.*` | `vacation.policy.name.required` |
| 파일 | `error.file.*` | `error.file.notfound` |
| 공통 | `error.common.*` | `error.common.invalid.input` |

### 주의사항
- **ErrorCode의 messageKey는 반드시 messages.properties에 존재해야 함**
- **MessageKey의 key도 반드시 messages.properties에 존재해야 함**
- 메시지 추가/수정 시 3개 언어 파일 모두 동기화 필수
- 사용하지 않는 메시지 키는 정기적으로 정리

## Exception 처리

### 예외 처리 원칙
- **Unchecked Exception (RuntimeException) 사용**: `@Transactional` 자동 롤백 지원
- **전역 처리**: `GlobalExceptionHandler`에서 일괄 처리, Service에서 try-catch 금지
- **ErrorCode 필수**: 모든 비즈니스 예외는 ErrorCode를 포함하여 throw

### 예외 클래스 계층 구조
```
RuntimeException
  └── BusinessException (기본 비즈니스 예외)
      ├── EntityNotFoundException      # DB 엔티티 조회 실패 (404)
      ├── InvalidValueException        # 입력값 검증 실패 (400)
      ├── DuplicateException           # 중복 데이터 (409)
      ├── BusinessRuleViolationException # 비즈니스 규칙 위반 (400)
      ├── ForbiddenException           # 권한 없음 (403)
      ├── UnauthorizedException        # 인증 실패 (401)
      ├── ExternalServiceException     # 외부 서비스 연동 실패 (502/503)
      └── ResourceNotFoundException    # 파일 등 리소스 조회 실패 (404)
```

### 예외 사용 예시
```java
// 엔티티 조회 실패
throw new EntityNotFoundException(ErrorCode.USER_NOT_FOUND);

// 입력값 검증 실패
throw new InvalidValueException(ErrorCode.VACATION_INVALID_DATE);

// 중복 데이터
throw new DuplicateException(ErrorCode.USER_ALREADY_EXISTS);

// 비즈니스 규칙 위반
throw new BusinessRuleViolationException(ErrorCode.VACATION_INSUFFICIENT_BALANCE);

// 외부 서비스 실패 (원인 예외 포함)
throw new ExternalServiceException(ErrorCode.INTERNAL_SERVER_ERROR, "메일 발송 실패", cause);

// 커스텀 메시지 사용
throw new EntityNotFoundException(ErrorCode.USER_NOT_FOUND, "사용자 ID: " + userId);
```

### 예외 생성자 (모든 예외 클래스 공통)
```java
XxxException(ErrorCode errorCode)                                    // 기본
XxxException(ErrorCode errorCode, String customMessage)              // 커스텀 메시지
XxxException(ErrorCode errorCode, Throwable cause)                   // 원인 예외
XxxException(ErrorCode errorCode, String customMessage, Throwable cause) // 둘 다
```

### 주의사항
- **Service에서 try-catch 사용 금지** (IOException 등 체크드 예외 변환 시에만 허용)
- **RuntimeException 직접 사용 금지** → 적절한 비즈니스 예외로 래핑
- 체크드 예외 발생 시 `ExternalServiceException`으로 래핑하여 throw

## API Guidelines
- Base path: /api/v1 [web:27]
- 에러 코드: 도메인별 접두어(USER_*, ORDER_*), HTTP 상태와 매핑 [web:26]

## Persistence Rules
- Soft delete: 공통 Audit + soft delete 필드 사용 (is_deleted) [web:27]
- N+1 방지: fetch join 또는 EntityGraph, 필요 시 read-only projection [web:26]
- 트랜잭션: 서비스 계층에서만 시작, 중첩 금지 [web:26]

## Testing Strategy
- 테스트 대상: Repository, Service 레이어만 테스트 (Controller 테스트 제외)
- Unit: JUnit 5 + Mockito, 빠른 실행
- Integration: @SpringBootTest + Testcontainers(Postgres)
- 테스트 데이터: Fixture/Factory 패턴, 임의 값은 Instancio/JavaFaker
- Repository/Service 변경 시 관련 테스트 코드도 반드시 함께 생성/수정/삭제
- **Jacoco 커버리지 목표: 90% 이상 달성 필수**
  - 모든 public 메서드에 대한 테스트 케이스 작성
  - 정상 케이스 + 예외 케이스(Exception 발생 경로) 모두 커버
  - 분기문(if/else, switch)의 모든 경로 테스트
  - 경계값 테스트 포함 (null, empty, 최대/최소값 등)

## Build & Run
- Gradle
    - build: ./gradlew clean build -x test [web:26]
    - test: ./gradlew test integrationTest [web:26]
    - run: ./gradlew bootRun -Dspring.profiles.active=local [web:26]

## Observability
- Metrics: Micrometer + Prometheus, 핵심 비즈 지표 계측 [web:26]
- Tracing: OpenTelemetry 자동 계측 + TraceId 로깅 상관관계 [web:26]
- Health: /actuator/health, readiness/liveness 분리 [web:26]

## Do Not
- 엔티티 직렬화로 API 응답 반환 금지 [web:26]
- EAGER 남발 금지, 양방향 연관 복잡화 금지 [web:26]
- 비밀값/토큰/키를 코드나 로그에 노출 금지 [web:26]

---
> Source: [lshdainty/porest-hr-back](https://github.com/lshdainty/porest-hr-back) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-15 -->
