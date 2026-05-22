## product-service

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build & Development Commands

```bash
# Build
./gradlew clean build

# Run ALL tests (2470 tests across 5 modules)
./gradlew test

# Run tests by module
./gradlew :service:test             # all service + global tests
./gradlew :app:test                 # application context, benchmark

# Run single test class
./gradlew :service:test --tests "*.Oauth2ControllerTest"

# Run single test method
./gradlew :service:test --tests "*.Oauth2ControllerTest.getOauth2LoginUri"

# Parallel build
./gradlew test --parallel

# Run application
./gradlew bootRun                                    # Default (port 8443)
./gradlew bootRun --args='--spring.profiles.active=test'  # Test profile (port 18080)

# Generate API documentation
./gradlew openapi3 && ./gradlew sortOpenApiJson && ./gradlew copySortedOpenApiJson

# Generate GraphQL classes from DGS schema
./gradlew generateJava

# Test coverage report (minimum 70%)
./gradlew test jacocoTestReport
# Report: app/build/reports/jacoco/html/index.html
```

## Architecture Overview

**Multi-Service Monolith**: Spring Boot 3.4.5 application organized as a Gradle multi-module project (2 modules +
composite build). Services share a single deployment unit but use separate databases per service. Designed for future
MSA migration with Saga pattern.

### Gradle Multi-Module Structure

```
product-service/
├── settings.gradle                    # 2 modules: service, app + includeBuild platform
├── build.gradle                       # Root: common settings, BOM, -parameters flag
├── service/build.gradle               # ALL 12 services + global infra (single compilation, multi-srcDirs)
│   ├── src/main/java/                 # Global infra (datasource, security, profanity, translation 등)
│   ├── shared-test/src/test/          # Shared test utils (ControllerTestConfig, MockUtil)
│   ├── user-service/src/main/java/
│   ├── guild-service/src/main/java/
│   ├── ... (12 directories)
│   └── support-service/src/main/java/
└── app/build.gradle                   # Bootstrap + DGS codegen + JaCoCo
    ├── src/main/java/                 # LevelUpTogetherMvpApplication.java
    ├── src/main/resources/            # All config files, schemas, keystores
    └── src/test/java/                 # @SpringBootTest tests only (3 files)
```

**Platform shared library** (`../level-up-together-platform`): `includeBuild`로 IDE에서 소스 편집 가능. CI에서는 GitHub Packages
Maven artifact 사용.

- `lut-platform-kernel` — 순수 공유 타입, audit entity, API result, enums
- `lut-platform-infra` — 공통 Spring infra (Redis, security, handler 등)
- `lut-platform-saga` — Saga 프레임워크 + SagaDataSourceConfig

**Why single service module**: Circular dependencies between services (user↔guild, user↔gamification, user↔support,
guild↔gamification) prevent independent Gradle modules. Directories are separated for logical boundaries, compiled as
one unit via `sourceSets.main.java.srcDirs`.

### Service Modules (`service/{name}/src/main/java/io/pinkspider/leveluptogethermvp/`)

| Service               | Database        | Purpose                                                                                                            |
|-----------------------|-----------------|--------------------------------------------------------------------------------------------------------------------|
| `userservice`         | user_db         | OAuth2 authentication (Google, Kakao, Apple), JWT tokens, profiles, friends, quests                                |
| `missionservice`      | mission_db      | Mission definition, progress tracking, Saga orchestration, mission book, daily mission instances (pinned missions) |
| `guildservice`        | guild_db        | Guild creation/management, members, experience/levels, bulletin board, territory, invitations                      |
| `chatservice`         | chat_db         | Guild chat messaging, chat participants, read status, direct messages                                              |
| `metaservice`         | meta_db         | Common codes, calendar holidays, Redis-cached metadata, level configuration, attendance reward configuration       |
| `feedservice`         | feed_db         | Activity feed (CQRS Read Model), likes, comments, feed visibility, FeedProjectionEventListener                     |
| `notificationservice` | notification_db | Push notifications, notification preferences, notification management                                              |
| `adminservice`        | admin_db        | Home banners, featured content (players, guilds, feeds)                                                            |
| `gamificationservice` | gamification_db | Titles, achievements, user stats, experience/levels, attendance tracking, events, seasons                          |
| `bffservice`          | -               | Backend-for-Frontend API aggregation, unified search                                                               |
| `noticeservice`       | -               | Notice/announcement management (layered: api, application, core, domain)                                           |
| `supportservice`      | -               | Customer support (1:1 inquiry) + Report handling (`/api/v1/reports`, ReportService → Admin Backend Feign)         |

### Global Infrastructure (`service/src/main/java/io/pinkspider/global/`)

MVP 전용 인프라 코드 (platform 공유 라이브러리와 별도):

- **Multi-datasource**: All 9 DataSourceProperties + DataSourceConfigs (`global.config.datasource`)
- **Security**: SecurityConfig, OAuth2Properties, CurrentUserArgumentResolver
- **Config**: HibernateConfig, RateLimiterConfig, WebMvcConfig, WebSocketConfig, FirebaseConfig, **ShedLockConfig**
- **i18n**: MessageConfig, LocaleInterceptor, i18n properties (errors/notifications ko/en/ja)
- **Translation**: Google Translation API integration (`global.translation`)
- **Profanity**: Profanity detection and validation with locale support (`leveluptogethermvp/profanity/`)
- **Image Moderation**: ONNX-based NSFW 이미지 검증 + AOP (`global.moderation`)
- **Image Storage**: S3 + CloudFront CDN (`global.config.s3`) — prod에서 S3 업로드, dev/test에서 로컬 파일시스템
- **Messaging**: AppPushMessageProducer (Redis Streams) — 인앱 알림 비동기 발행
- **Rate Limiting**: PerUserRateLimit + PerUserRateLimitAspect
- **GraphQL**: DGS context, scalars, fetchers
- **Feign**: AdminInternalFeignClient (Admin Backend 연동)
- **Distributed Lock**: ShedLock + Redis (`@SchedulerLock`) — 멀티 EC2 인스턴스 스케줄러 동시 실행 방지

### Platform Shared Library (`level-up-together-platform` 별도 레포)

`includeBuild`로 IDE에서 소스 편집 가능. 공통 인프라:

- **kernel**: ApiResult, ApiStatus, CustomException, Base Entity, Domain Events, Enums, Utils, **Facade 인터페이스** (
  UserQueryFacade, GuildQueryFacade, GamificationQueryFacade)
- **infra**: RedisConfig, AsyncConfig, JpaAuditingConfig, QueryDslConfig, JwtAuthenticationFilter, RestExceptionHandler,
  CryptoConverter
- **saga**: SagaOrchestrator, AbstractSagaStep, SagaDataSourceConfig

### Transaction Manager (Critical)

멀티 데이터소스 환경에서 `userTransactionManager`가 `@Primary`로 설정되어 있으므로, **각 서비스의 `@Transactional`에 명시적으로 트랜잭션 매니저를 지정해야 함**:

```java
// GuildService 예시
@Transactional(transactionManager = "guildTransactionManager")
public void updateGuild(...) { ...}

// MissionService 예시
@Transactional(transactionManager = "missionTransactionManager")
public void updateMission(...) { ...}
```

| Service             | Transaction Manager                |
|---------------------|------------------------------------|
| userservice         | `userTransactionManager` (Primary) |
| missionservice      | `missionTransactionManager`        |
| guildservice        | `guildTransactionManager`          |
| chatservice         | `chatTransactionManager`           |
| metaservice         | `metaTransactionManager`           |
| feedservice         | `feedTransactionManager`           |
| notificationservice | `notificationTransactionManager`   |
| adminservice        | `adminTransactionManager`          |
| gamificationservice | `gamificationTransactionManager`   |
| saga                | `sagaTransactionManager`           |

### Service Layer Pattern

Each service module follows a consistent layered structure:

- `api/` - REST controllers returning `ApiResult<T>` wrapper
- `application/` - Business logic services with `@Transactional`
- `core/` - Core domain logic (some services use this instead of or alongside `application/`)
- `domain/` - Entities, DTOs, enums
- `infrastructure/` - JPA repositories
- `scheduler/` - Scheduled batch jobs (optional)
- `saga/` - Saga orchestration steps (optional)

Note: Some services vary slightly (e.g., `noticeservice`/`supportservice` use `core/` instead of `application/`).
`feedservice` follows CQRS pattern with `FeedQueryService` (read) + `FeedCommandService` (write).

### Cross-Service Boundary Rules (MSA 준비)

**다른 서비스의 DB에 직접 접근(Repository import) 금지** — 반드시 Facade 인터페이스를 통해 접근:

```java
// BAD — 다른 서비스의 Repository 또는 Service 직접 사용
@Service
public class MyPageService {

    private final UserTitleRepository userTitleRepository; // gamification_db 직접 접근
    private final TitleService titleService;               // 구체 서비스 직접 의존
}

// GOOD — Facade 인터페이스를 통해 접근
@Service
public class MyPageService {

    private final GamificationQueryFacade gamificationQueryFacade;
}
```

**Facade 인터페이스** (`lut-platform-kernel`에 정의, 각 서비스에서 구현):

| Facade                    | 구현체                              | 주요 용도                   |
|---------------------------|----------------------------------|-------------------------|
| `UserQueryFacade`         | `UserQueryFacadeService`         | 프로필, 닉네임, 친구 관계, 존재 확인  |
| `GuildQueryFacade`        | `GuildQueryFacadeService`        | 길드 정보, 멤버십, 권한 체크, 경험치  |
| `GamificationQueryFacade` | `GamificationQueryFacadeService` | 레벨, 칭호, 업적, 통계, 경험치, 시즌 |

**Facade DTO**: `io.pinkspider.global.facade.dto` 패키지에 서비스 간 전달용 DTO 정의
(예: `UserProfileInfo`, `GuildBasicInfo`, `TitleInfoDto`, `SeasonDto` 등 22개)

**현재 적용 완료:**

- 전체 서비스 간 직접 의존 → Facade 전환 완료 (Phase 3~5)
- Entity/Enum import는 현행 유지 (MSA 전환 시 DTO/common 라이브러리로 교체 예정)

### API Response Format

All REST endpoints return `ApiResult<T>` from `io.pinkspider.global.api`:

```json
{
  "code": "000000",
  "message": "success",
  "value": {
    ...
  }
}
```

### Exception Handling (i18n)

Custom exceptions use message keys resolved via `MessageSource` at response time:

```java
// 메시지 키 사용 (i18n properties에서 locale별 메시지 조회)
throw new CustomException("USER_001", "error.user.not_found");
// Accept-Language: en → "User not found."
// Accept-Language: ko → "사용자를 찾을 수 없습니다."
```

`RestExceptionHandler.resolveMessage()`가 `CustomException.message`를 키로 시도, 없으면 원문 반환.

**ApiStatus 코드 규칙** (6자리: 서비스 2자리 + 카테고리 2자리 + 일련번호 2자리):
| 서비스 | 코드 접두사 |
|--------|------------|
| global | 00 |
| bff-service | 01 |
| api-gateway | 02 |
| user-service | 03 |
| guild-service | 04 |
| mission-service | 05 |
| app-push-service | 06 |
| payment-service | 07 |
| meta-service | 08 |
| logger-service | 09 |
| stats-service | 10 |
| batch-service | 11 |
| gamification-service | 12 |
| feed-service | 13 |
| notification-service | 14 |

## Testing

### Test Distribution (2470 tests across 5 modules)

| Module            | Tests | Content                                              |
|-------------------|-------|------------------------------------------------------|
| `platform:kernel` | 39    | util tests                                           |
| `platform:infra`  | 168   | resolver, validation, profanity, crypto, translation |
| `platform:saga`   | 29    | saga framework tests                                 |
| `service`         | 2222  | all service unit + controller tests (multi-srcDirs)  |
| `app`             | 12    | ApplicationTests, benchmark, TestDataSourceConfig    |

### Shared Test Utilities

- `kernel/src/testFixtures/` → `TestReflectionUtils` (shared via `java-test-fixtures` plugin)
- `service/shared-test/src/test/java/` → `TestApplication`, `ControllerTestConfig`, `BaseTestController`, `MockUtil`
- `service/shared-test/src/test/resources/application.yml` → `spring.cloud.config.enabled: false`

### Test Fixtures

JSON fixtures in `service/{name}/src/test/resources/fixture/{servicename}/` loaded via `MockUtil`:

```java
MockUtil.readJsonFileToClass("fixture/userservice/oauth/mockCreateJwtResponseDto.json",CreateJwtResponseDto .class);
```

### Controller Tests (in `:service` module)

```java
@WebMvcTest(controllers = YourController.class, excludeAutoConfiguration = {...})
@Import(ControllerTestConfig.class)
@AutoConfigureRestDocs
@AutoConfigureMockMvc(addFilters = false)
@ActiveProfiles("test")
```

### Unit Tests (Service Layer, in `:service` module)

```java

@ExtendWith(MockitoExtension.class)
class YourServiceTest {

    @Mock
    private YourRepository repository;

    @InjectMocks
    private YourService service;

    // Reflection으로 엔티티 ID 설정 (JPA 엔티티는 ID가 auto-generated)
    private void setEntityId(Entity entity, Long id) {
        try {
            Field idField = Entity.class.getDeclaredField("id");
            idField.setAccessible(true);
            idField.set(entity, id);
        } catch (Exception e) {
            throw new RuntimeException(e);
        }
    }
}
```

### Integration Tests (in `:app` module — needs full application context)

```java

@SpringBootTest
@ActiveProfiles("test")
@Transactional(transactionManager = "yourTransactionManager")
class YourIntegrationTest {

    @Autowired
    private YourService service;
}
```

## 코드 컨벤션

- **API 필드명**: 프론트와 백엔드간 통신은 snake_case 사용
- **들여쓰기**: 4 spaces
- **DTO**: record 사용 권장
- **레이어 구조**: Controller → Service → Repository
- **테스트**: JUnit 5 + Mockito
- **날짜/시간**: ISO 8601 (`2026-03-24T14:30:45`), UTC 기준 저장
- **에러 메시지**: i18n 메시지 키 사용 (`error.xxx.yyy`), 한국어 하드코딩 금지
- **기본 언어**: 영어 (Default)

## Internationalization (i18n)

### 타임존

- JVM: `TimeZone.setDefault(UTC)` (`@PostConstruct`)
- Hibernate: `hibernate.jdbc.time_zone: UTC`
- Jackson: `LmObjectMapper.setTimeZone(UTC)` + `spring.jackson.time-zone: UTC`
- JDBC URL: `TimeZone=UTC`
- 스케줄러: `@Scheduled(cron = "...", zone = "Asia/Seoul")` — 비즈니스 시간대 명시

### 메시지 번역 (MessageSource)

i18n properties 파일: `app/src/main/resources/i18n/`

```
i18n/
├── errors_ko.properties        # 에러 메시지 (한국어)
├── errors_en.properties        # 에러 메시지 (영어)
├── errors_ja.properties        # 에러 메시지 (일본어)
├── notifications_ko.properties # 알림 메시지 (한국어)
├── notifications_en.properties # 알림 메시지 (영어)
├── notifications_ja.properties # 알림 메시지 (일본어)
├── messages_ko.properties      # 일반 메시지 (한국어)
├── messages_en.properties      # 일반 메시지 (영어)
└── messages_ja.properties      # 일반 메시지 (일본어)
```

- `MessageConfig.java`: `ReloadableResourceBundleMessageSource` Bean (UTF-8, `useCodeAsDefaultMessage=true`)
- `LocaleInterceptor`: `Accept-Language` 헤더 → `LocaleContextHolder` 설정
- `RestExceptionHandler.resolveMessage()`: `CustomException.message`를 키로 MessageSource 조회

### 콘텐츠 번역 (Google Translation API)

- `TranslationService`: 3-tier 캐시 (Redis 7일 → DB → Google API)
- Feed, Guild 게시판/댓글 조회 시 `Accept-Language` 헤더로 on-demand 번역
- `ContentType` enum: `FEED`, `FEED_COMMENT`, `GUILD_POST`, `GUILD_COMMENT`
- 설정: `google.translation.enabled: true/false`

### 유저 언어 설정

- `Users.preferredLocale`: `VARCHAR(5) DEFAULT 'en'`
- API: `PUT /api/v1/mypage/preferred-locale`
- 푸시 알림 발송 시 유저 locale 조회 → MessageSource로 해당 언어 메시지 생성

### 금칙어 (Profanity)

- `ProfanityWord.locale`: 언어별 금칙어 관리 (`ko`, `en`, `ar`, `ja`)
- Unique constraint: `(locale, word)`
- 전체 언어 통합 검사 + locale별 검사 모두 지원

### 글로벌 서비스 로드맵

전체 마이그레이션 계획: `docs/GLOBAL_SERVICE_ROADMAP.md`

## Event-Driven 패턴

### 이벤트 발행

```java

@Service
@RequiredArgsConstructor
public class YourService {

    private final ApplicationEventPublisher eventPublisher;

    @Transactional(transactionManager = "yourTransactionManager")
    public void doSomething() {
        // 비즈니스 로직
        eventPublisher.publishEvent(new YourEvent(userId, data));
    }
}
```

### 이벤트 수신

```java

@Component
@RequiredArgsConstructor
public class YourEventListener {

    @TransactionalEventListener(phase = TransactionPhase.AFTER_COMMIT)
    public void handleEvent(YourEvent event) {
        // 이벤트 처리 (다른 서비스 호출, 알림 발송 등)
    }
}
```

### 주요 이벤트 흐름

| 발행 서비스                 | 이벤트                                | 수신 리스너                               | 처리 내용                                  |
|------------------------|------------------------------------|--------------------------------------|----------------------------------------|
| GuildService           | `GuildJoinedEvent`                 | `AchievementEventListener`           | 길드 가입 업적 체크                            |
| GuildService           | `GuildJoinedEvent`                 | `FeedProjectionEventListener`        | 길드 가입 피드 생성                            |
| GuildService           | `GuildCreatedEvent`                | `FeedProjectionEventListener`        | 길드 창설 피드 생성                            |
| GuildService           | `GuildInvitationEvent`             | `NotificationEventListener`          | 초대 알림 발송                               |
| GuildExperienceService | `GuildLevelUpEvent`                | `FeedProjectionEventListener`        | 길드 레벨업 피드 생성                           |
| FriendService          | `FriendRequestAcceptedEvent`       | `NotificationEventListener`          | 친구 수락 알림                               |
| FriendService          | `FriendRequestAcceptedEvent`       | `FeedProjectionEventListener`        | 친구 추가 피드 생성 (양쪽)                       |
| FriendService          | `FriendRequestAcceptedEvent`       | `UserStatsCounterEventListener`      | friendCount 증가 + 업적 체크                 |
| FriendService          | `FriendRemovedEvent`               | `UserStatsCounterEventListener`      | friendCount 감소 (양쪽)                    |
| GamificationService    | `TitleAcquiredEvent`               | `NotificationEventListener`          | 칭호 획득 알림                               |
| GamificationService    | `TitleAcquiredEvent`               | `FeedProjectionEventListener`        | 칭호 획득 피드 생성                            |
| GamificationService    | `AchievementCompletedEvent`        | `NotificationEventListener`          | 업적 달성 알림                               |
| GamificationService    | `AchievementCompletedEvent`        | `FeedProjectionEventListener`        | 업적 달성 피드 생성                            |
| GamificationService    | `TitleEquippedEvent`               | `FeedProjectionEventListener`        | 칭호 변경 피드 업데이트                          |
| UserExperienceService  | `UserLevelUpEvent`                 | `FeedProjectionEventListener`        | 레벨업 피드 생성                              |
| UserExperienceService  | `UserLevelUpEvent`                 | `UserLevelUpProfileSyncListener`     | 유저 프로필 레벨 동기화                          |
| AttendanceService      | `AttendanceStreakEvent`            | `FeedProjectionEventListener`        | 연속 출석 피드 생성                            |
| MissionService         | `MissionStateChangedEvent`         | `MissionStateHistoryEventListener`   | 미션 상태 이력 저장                            |
| GuildMemberService     | `GuildMemberJoinedChatNotifyEvent` | `ChatEventListener`                  | 채팅방 입장 알림                              |
| GuildMemberService     | `GuildMemberLeftChatNotifyEvent`   | `ChatEventListener`                  | 채팅방 퇴장 알림                              |
| GuildMemberService     | `GuildMemberKickedChatNotifyEvent` | `ChatEventListener`                  | 채팅방 추방 알림                              |
| UserService            | `UserSignedUpEvent`                | `UserSignedUpEventListener`          | 기본 칭호 부여                               |
| UserService            | `UserProfileChangedEvent`          | `*ProfileSnapshotEventListener` (x4) | 비정규화 닉네임 동기화 (chat/feed/guild/mission) |
| FeedCommandService     | `FeedLikedEvent`                   | `UserStatsCounterEventListener`      | likesReceived 증가 + 업적 체크               |
| FeedCommandService     | `FeedUnlikedEvent`                 | `UserStatsCounterEventListener`      | likesReceived 감소                       |
| GuildService           | `GuildJoinedEvent`                 | `UserStatsCounterEventListener`      | guildJoinCount 증가 + 업적 체크              |
| MissionCompletionSaga  | `MissionCompletedCountEvent`       | `UserStatsCounterEventListener`      | totalMissionCompletions 증가 + 업적 체크     |
| MissionCompletionSaga  | `GuildMissionCompletedCountEvent`  | `UserStatsCounterEventListener`      | totalGuildMissionCompletions 증가 + 업적 체크 |
| MissionAutoCompleteScheduler | `MissionAutoEndWarningEvent` | `NotificationEventListener`          | 미션 자동 종료 10분 전 경고 알림                  |

> **자동 피드 생성 축소 (QA-35)**: 칭호 획득/업적 달성/길드 가입/친구 추가 피드는 비활성화. 레벨업/길드 레벨업 피드는 **10단위 마일스톤**(Lv 10, 20, 30…)에서만 생성.

## Redis Caching

| 캐시 서비스                    | 캐시 키                             | TTL |
|---------------------------|----------------------------------|-----|
| `UserProfileCacheService` | `userProfile:{userId}`           | 5분  |
| `FriendCacheService`      | `friendIds:{userId}`             | 10분 |
| `TitleService`            | `userTitleInfo:{userId}`         | 5분  |
| `MissionCategoryService`  | `missionCategories:{categoryId}` | 1시간 |

## Scheduler & Distributed Lock (ShedLock)

멀티 EC2 인스턴스 환경에서 `@Scheduled` 메서드의 동시 실행을 막기 위해 **모든 스케줄러는 `@SchedulerLock` 필수**. `ShedLockConfig`가 Redis SETNX 기반 LockProvider를 등록 (`prefix=lut`). `SchedulerLockCoverageTest`가 누락된 락을 컴파일 타임에 잡아낸다.

```java
@Scheduled(cron = "0 0 0 * * *", zone = "Asia/Seoul")
@SchedulerLock(name = "MyScheduler_method", lockAtMostFor = "PT10M", lockAtLeastFor = "PT1M")
public void run() { ... }
```

| 스케줄러 | 주기/Zone | Lock 이름 | 비고 |
|--------|---------|---------|------|
| `DailyMissionInstanceScheduler.generateDailyInstances` | `0 0 0 * * *` KST | `DailyMissionInstanceScheduler_generateDailyInstances` | 고정 미션 일일 인스턴스 생성 + 자정 자동완료 |
| `MissionAutoCompleteScheduler.autoCompleteExpiredMissions` | 5분 fixedRate | `MissionAutoCompleteScheduler_autoCompleteExpiredMissions` | 만료 미션 자동 종료 + 10분 전 경고 알림 |
| `TokenMaintenanceScheduler.cleanupExpiredSessions` | `0 0 2 * * *` KST | `TokenMaintenanceScheduler_cleanupExpiredSessions` | 만료된 OAuth 세션 정리 |
| `TokenMaintenanceScheduler.cleanupOrphanedUserSessions` | `0 30 2 * * *` KST | `TokenMaintenanceScheduler_cleanupOrphanedUserSessions` | 고아 user_sessions 참조 정리 |
| `DailyMvpHistoryScheduler.saveDailyMvpHistory{Kst,Ast,Utc}` | `0 0 0 * * *` (3 zones: KST/AST/UTC) | `DailyMvpHistoryScheduler_{kst,ast,utc}` | 타임존별 일간 MVP 기록 |
| `SeasonRewardScheduler.processEndedSeasonRewards` | `0 0 3 * * *` KST | `SeasonRewardScheduler_processEndedSeasonRewards` | 종료된 시즌 보상 자동 부여 |

## Saga Pattern (Mission Completion 예시)

```
missionservice/saga/
├── MissionCompletionSaga.java        - Saga 오케스트레이터 (Regular + Pinned 분기)
├── MissionCompletionContext.java     - Saga 컨텍스트 (공유 데이터, isPinned 플래그)
└── steps/
    ├── LoadMissionDataStep.java           - 일반 미션 데이터 로드
    ├── LoadPinnedMissionDataStep.java     - 고정 미션 데이터 로드
    ├── CompleteExecutionStep.java         - 일반 미션 실행 완료 (2시간 EXP 차등)
    ├── CompletePinnedInstanceStep.java    - 고정 미션 인스턴스 완료
    ├── CreateNextPinnedInstanceStep.java  - 다음 고정 미션 인스턴스 생성
    ├── UpdateParticipantProgressStep.java - 참가자 진행 상태 업데이트
    ├── GrantUserExperienceStep.java       - 유저 경험치 지급
    ├── GrantGuildExperienceStep.java      - 길드 경험치 지급
    ├── UpdateUserStatsStep.java           - 유저 통계 업데이트
    └── CreateFeedFromMissionStep.java     - 피드 생성
```

### Saga Step 구현

```java

@Component
public class YourStep extends AbstractSagaStep<YourContext> {

    @Override
    public String getStepName() {
        return "YOUR_STEP";
    }

    @Override
    protected SagaStepResult executeInternal(YourContext context) {
        // 비즈니스 로직
        return SagaStepResult.success();
    }

    @Override
    protected SagaStepResult compensateInternal(YourContext context) {
        // 보상 트랜잭션 (롤백 로직)
        return SagaStepResult.success();
    }
}
```

## HTTP API 테스트

`http/` 폴더에 IntelliJ HTTP Client 형식의 API 테스트 파일:

| 파일                     | 설명                                |
|------------------------|-----------------------------------|
| `oauth-jwt.http`       | OAuth2 로그인, JWT 토큰 관리, 모바일 소셜 로그인 |
| `mission.http`         | 미션 CRUD, 참가자, 실행 추적, 캘린더          |
| `guild.http`           | 길드 관리, 게시판, 거점                    |
| `guild-chat.http`      | 길드 채팅 (메시지, 참여자, 읽음)              |
| `activity-feed.http`   | 피드, 좋아요, 댓글, 검색                   |
| `friend.http`          | 친구 요청/수락/거절/차단                    |
| `mypage.http`          | 프로필, 닉네임, 칭호 관리                   |
| `achievement.http`     | 업적, 칭호, 레벨 랭킹                     |
| `attendance.http`      | 출석 체크                             |
| `notification.http`    | 알림 관리, 읽음 처리                      |
| `device-token.http`    | FCM 토큰 등록/삭제                      |
| `event.http`           | 이벤트 API                           |
| `bff.http`             | BFF 홈, 통합 검색, 시즌                  |
| `home.http`            | 홈 배너, 추천 콘텐츠                      |
| `meta.http`            | 메타데이터, 공통 코드                      |
| `user-terms.http`      | 약관 동의                             |
| `user-experience.http` | 경험치, 레벨                           |
| `guild-dm.http`        | 길드 DM (다이렉트 메시지)                  |
| `test-login.http`      | 테스트 로그인                           |

환경 설정: `http/http-client.env.json`

```json
{
  "dev": {
    "baseUrl": "https://dev-api.level-up-together.com"
  },
  "local": {
    "baseUrl": "https://local.level-up-together.com:8443"
  },
  "test": {
    "baseUrl": "http://localhost:18080"
  }
}
```

## Configuration Profiles

| 프로필                                            | 설명                                    |
|------------------------------------------------|---------------------------------------|
| `application.yml`                              | Default configuration                 |
| `application-test.yml`                         | H2 databases, test Kafka (port 18080) |
| `application-unit-test.yml`                    | Unit test configuration               |
| `application-push-test.yml`                    | Push notification test configuration  |
| `application-local.yml`                        | Config server integration             |
| `application-dev.yml` / `application-prod.yml` | Environment-specific                  |

Note: Config files are located in `app/src/main/resources/config/`, not the root of `resources/`.

## 관련 프로젝트

| 프로젝트              | 경로                                                                                  |
|-------------------|-------------------------------------------------------------------------------------|
| Admin Backend     | `/Users/pink-spider/Code/github/Level-Up-Together/admin-service`                    |
| Admin Frontend    | `/Users/pink-spider/Code/github/Level-Up-Together/level-up-together-admin-frontend` |
| Product Backend   | `/Users/pink-spider/Code/github/Level-Up-Together/product-service`                  |
| Product Frontend  | `/Users/pink-spider/Code/github/Level-Up-Together/level-up-together-frontend`       |
| SQL Scripts       | `/Users/pink-spider/Code/github/Level-Up-Together/level-up-together-sql/queries`    |
| Config Server     | `/Users/pink-spider/Code/github/Level-Up-Together/config-server`                    |
| Config Repo       | `/Users/pink-spider/Code/github/Level-Up-Together/config-repository`                |
| Service Discovery | `/Users/pink-spider/Code/github/Level-Up-Together/service-discovery`                |
| React Native App  | `/Users/pink-spider/Code/github/Level-Up-Together/LevelUpTogetherReactNative`       |

## 자주 발생하는 이슈

### QueryDSL 빌드 오류

`Attempt to recreate a file for type Q*` 오류 발생 시:

```bash
./gradlew clean compileJava
```

### 트랜잭션 매니저 미지정 오류

데이터가 저장되지 않거나 조회되지 않는 경우, `@Transactional`에 올바른 트랜잭션 매니저가 지정되어 있는지 확인

### Integration Tests 실패

SSH 터널이나 외부 서비스 연결이 필요한 테스트는 로컬에서 실패할 수 있음. `@ActiveProfiles("test")` 확인

### Race Condition (중복 키 오류) 해결 패턴

동시 요청으로 인한 `DataIntegrityViolationException` (Unique constraint violation) 발생 시:

```java
// Check-then-insert 패턴 대신 saveAndFlush + 예외 처리 사용
private Entity getOrCreateEntity(String key) {
    return repository.findByKey(key)
        .orElseGet(() -> {
            try {
                Entity newEntity = Entity.builder().key(key).build();
                return repository.saveAndFlush(newEntity);  // 즉시 INSERT
            } catch (DataIntegrityViolationException e) {
                // 중복 발생 시 기존 레코드 조회
                return repository.findByKey(key)
                    .orElseThrow(() -> new IllegalStateException("Race condition 처리 실패"));
            }
        });
}
```

적용 사례: `AchievementService.getOrCreateUserAchievement()`, `AttendanceService.checkIn()`,
`NotificationService.createNotificationWithDeduplication()`

## Image Moderation (이미지 검증)

ONNX Runtime 기반 NSFW 이미지 자동 검증 시스템 (`global.moderation`):

### 아키텍처: Strategy Pattern + AOP

- `@ModerateImage` 어노테이션을 메서드에 적용하면 `MultipartFile` 파라미터를 자동 탐색하여 검증
- `ImageModerationAspect`가 `@Around` 어드바이스로 검증 실행
- `ModerationConfig`가 `moderation.image.provider` 설정에 따라 구현체 선택

### Provider 구현체

| Provider          | 클래스                               | 설명                               |
|-------------------|-----------------------------------|----------------------------------|
| `none` (기본값)      | `NoOpImageModerationService`      | 비활성화 (dev/test 환경)               |
| `onnx-nsfw`       | `OnnxNsfwModerationService`       | ONNX Runtime + OpenNSFW2 모델 ($0) |
| `aws-rekognition` | `AwsRekognitionModerationService` | AWS Rekognition (스켈레톤)           |

### 설정

```yaml
moderation:
  image:
    provider: onnx-nsfw   # none | onnx-nsfw | aws-rekognition
    onnx:
      model-path: classpath:models/nsfw.onnx
      nsfw-threshold: 0.8
```

### 적용된 서비스

- `GuildService` — 길드 이미지 업로드
- `MyPageService` — 프로필 이미지 업로드
- `EventController` — 이벤트 이미지 업로드
- `PinnedMissionExecutionStrategy` / `RegularMissionExecutionStrategy` — 미션 이미지

### 에러 코드

부적절 이미지 감지 시: `CustomException("000010", "error.moderation.inappropriate_image")`

## Image Storage (이미지 저장)

`@Profile` 기반 Strategy Pattern으로 환경별 이미지 저장소 분기:

| 환경          | 구현체                          | 저장소                                   |
|-------------|------------------------------|---------------------------------------|
| `prod`      | `S3*ImageStorageService`     | S3 (`lut-images-prod`) + CloudFront CDN |
| `!prod`     | `Local*ImageStorageService`  | 로컬 파일시스템 + Spring MVC 리소스 핸들러         |

### S3 구현체 (prod)

- `S3Config` — `S3Client` Bean (`@Profile("prod")`, EC2 IAM Role 자동 인증)
- `S3ImageProperties` — `app.upload.s3.bucket` + `app.upload.s3.cdn-base-url`
- S3 키 패턴: `profile/{userId}/{uuid}.ext`, `guild/{guildId}/{uuid}.ext`, `missions/{userId}/{missionId}/{date}_{uuid}.ext`, `events/{uuid}.ext`
- CDN URL 반환: `https://images.level-up-together.com/{key}`

### 서비스별 구현체

| 서비스 | S3 구현체 (prod)                  | Local 구현체 (!prod)                |
|------|-------------------------------|----------------------------------|
| 프로필  | `S3ProfileImageStorageService`  | `LocalProfileImageStorageService`  |
| 길드   | `S3GuildImageStorageService`    | `LocalGuildImageStorageService`    |
| 미션   | `S3MissionImageStorageService`  | `LocalMissionImageStorageService`  |
| 이벤트  | `S3EventImageStorageService`    | `LocalEventImageStorageService`    |

### 설정

```yaml
# application.yml (기본값)
app:
  upload:
    s3:
      bucket: ""
      cdn-base-url: ""

# product-service-prod.yml (Config Server)
app:
  upload:
    s3:
      bucket: lut-images-prod
      cdn-base-url: https://images.level-up-together.com
```

## Feature-Specific Implementation Notes

### Pinned Mission (고정 미션) - Template-Instance 패턴

고정 미션(`isPinned=true`)은 `DailyMissionInstance` 엔티티를 사용:

- 매일 자동 생성 (스케줄러: `DailyMissionInstanceScheduler`, cron: `0 5 0 * * *`)
- 미션 정보 스냅샷 저장 (미션 변경 시 과거 기록 보존)
- 다중 실행 지원 (시퀀스 번호로 일일 복수 완료 추적)

API 라우팅 — Strategy Pattern (`MissionExecutionStrategy` 인터페이스):

```java
// MissionExecutionService에서 Strategy 패턴으로 분기
// PinnedMissionExecutionStrategy  → DailyMissionInstanceService 위임
// RegularMissionExecutionStrategy → Saga 기반 처리
```

**Strategy 메서드** (8개): `startExecution`, `endExecution`, `cancelExecution`, `getActiveExecution`,
`shareExecutionToFeed`, `unshareExecutionFromFeed`, `updateExecutionNote`, `updateExecutionImage`

### Mission Execution Mode (수행 방식)

미션의 수행 방식을 결정하는 `MissionExecutionMode` enum:

```java
TIMED   // 시간 측정 (기본값) — 분당 1 EXP, 2시간 자동완료
SIMPLE  // 수행 여부 — 즉시 완료, 고정 +5 EXP, 하루 10회 제한
```

- `Mission.executionMode` 필드로 미션 생성 시 선택 (TIMED/SIMPLE)
- `MissionExecutionLifecycle.complete()`에서 SIMPLE 모드 분기
- SIMPLE 모드는 최소 수행 시간 제한 없음, 자동완료 불필요
- 하루 제한 체크: `MissionExecutionService.validateSimpleDailyLimit()`
  (일반+고정 미션의 SIMPLE 완료 합산)

### Feed Visibility at Execution (피드 공개범위)

피드 공개범위를 미션 생성이 아닌 **실행 완료 시** 선택:

- `MissionExecutionController.completeExecution()` — `feedVisibility` 파라미터 (PUBLIC/FRIENDS/PRIVATE)
- `Users.preferredFeedVisibility` — 유저의 최근 선택값을 기억 (쿠팡 UX 패턴)
- `GET/PUT /api/v1/mypage/preferred-feed-visibility` — 선호 공개범위 조회/수정
- Saga의 `CreateFeedFromMissionStep`은 `context.getFeedVisibility()`를 직접 사용
- **피드 중복 생성 방지**: `shareExecutionToFeed()`에서 기존 피드가 있으면 업데이트, 없으면 생성
  (`FeedCommandService.updateFeedContentByExecutionId()`)

### Feed Search Type (홈피드 필터)

홈 피드 조회 시 필터별 뷰:

```java
FeedSearchType { ALL, FRIENDS, GUILD, MINE }
```

- `GET /api/v1/feeds/public?searchType=FRIENDS` — 피드 API에 searchType 파라미터
- `GET /api/v1/bff/home?feedSearchType=GUILD` — BFF에 feedSearchType 파라미터
- `FeedQueryService.getFilteredFeeds()` — searchType별 분기
  - FRIENDS: `findFriendsFeeds()` (친구의 PUBLIC+FRIENDS 피드)
  - GUILD: `findGuildFeedsByGuildIds()` (내 길드들의 PUBLIC+GUILD 피드)
  - MINE: `findByUserId()` (내 피드 전체)

### Mission Feed Sync (미션 기록 ↔ 피드 동기화)

미션 실행의 note/image 변경 시 연관된 ActivityFeed에 자동 동기화:

- `MissionFeedNoteChangedEvent` → `FeedCommandService.updateFeedDescriptionByExecutionId()`
- `MissionFeedImageChangedEvent` → `FeedCommandService.updateFeedImageUrlByExecutionId()`
- `isSharedToFeed` 여부와 무관하게 항상 이벤트 발행 (Saga 생성 PRIVATE 피드도 동기화)

### Mission Template Propagation (템플릿 수정 전파)

어드민에서 `MissionTemplate`의 시간 설정을 수정하면 이미 복제된 Mission에도 전파:

- `MissionTemplateAdminService.updateTemplate()` → `MissionRepository.updateDurationByBaseMissionId()`
- `duration_minutes`, `target_duration_minutes` 일괄 업데이트 (`baseMissionId` 기반)

### Mission Soft Delete (미션 소프트 삭제)

미션 삭제 시 물리적 삭제 대신 소프트 삭제 사용:

- `Mission.isDeleted` (boolean) + `Mission.deletedAt` (LocalDateTime)
- 목록 조회 시 `isDeleted=false` 필터링

### Mission Auto-Complete (미션 자동 종료)

`MissionAutoCompleteScheduler` (5분 간격 실행):

- 목표 시간 도달 미션: 자동 완료 처리 (목표 시간 기준 EXP 지급)
- 2시간 초과 미션 (목표 시간 없는 경우): 자동 완료 (기본 EXP만 지급, `isAutoCompleted=true`)
- 자동 종료 10분 전: `MissionAutoEndWarningEvent` 발행 → 경고 알림 발송

### Guild Mission Auto-Enrollment (길드 미션 자동 참가)

길드원이 가입하거나 길드 미션이 공개되면 자동으로 참가자 등록:

- `MissionParticipantService.addGuildMemberAsParticipant()` — 중복 방지 포함
- 탈퇴/추방 시 참가자 정리

### Guild Invitation (길드 초대)

비공개 길드 초대 시스템:

- 초대 상태: `PENDING`, `ACCEPTED`, `REJECTED`, `CANCELLED`, `EXPIRED`
- 만료 시간: 7일
- 같은 카테고리의 다른 길드 가입자는 초대 불가

API 엔드포인트:

```
POST   /api/v1/guilds/{guildId}/invitations         - 초대 발송
GET    /api/v1/users/me/guild-invitations           - 내 대기중 초대 목록
POST   /api/v1/guild-invitations/{id}/accept        - 초대 수락
POST   /api/v1/guild-invitations/{id}/reject        - 초대 거절
DELETE /api/v1/guild-invitations/{id}               - 초대 취소 (마스터)
```

### Browse-First (비로그인 열람) — QA-110/QA-111

비로그인 상태에서도 홈/피드/길드의 GET 요청을 허용 (작성/수정/삭제는 인증 필요). `SecurityConfig`에서 `HttpMethod.GET`만 `permitAll`로 화이트리스트:

```
/api/v1/bff/home, /api/v1/bff/guild/list, /api/v1/bff/guild/{guildId}
/api/v1/feeds/public, /api/v1/feeds/{feedId}, /api/v1/feeds/{feedId}/comments
/api/v1/feeds/search, /api/v1/feeds/category/**
/api/v1/guilds/public, /api/v1/guilds/search, /api/v1/guilds/{guildId}
/api/v1/mypage/profile/{userId}, /api/v1/mypage/nickname/check
```

비인증 요청은 `Principal == null`이므로 컨트롤러/서비스에서 `currentUserId` Optional 처리 필요. 친구/좋아요/내 활동 등은 비로그인에서 빈 결과 또는 404 반환.

### Signup Token Flow (QA-108) — 회원가입 흐름

OAuth 콜백 시점에 `Users` row를 곧바로 INSERT하지 않고 **임시 signup token**으로 Redis 세션을 만든다. 닉네임/약관 동의가 모두 완료된 시점에 비로소 INSERT → 중도 이탈 시 잔여 row가 남지 않는다.

- `SignupTokenService` (TTL 30분, Redis)
  - `signup:{provider}:{emailHash}` → `SignupSessionData` JSON
  - `signup-token:{token}` → `{provider}:{emailHash}` 인덱스 (양방향)
- 클라이언트 흐름: OAuth 콜백 → `signup_token` 수신 → `POST /api/v1/auth/signup` (닉네임 + 약관 동의) → 정식 가입 + JWT 발급
- 신규/기존 사용자 응답 형태가 다름 (`SocialLoginResponseDto` 분기)
- 닉네임 중복 체크: `GET /api/v1/mypage/nickname/check` (비인증 허용)

### 신고 처리 워크플로우 (Report Processing)

`supportservice/report` — 사용자 신고 생성/조회 + 어드민 처리 결과 적용:

| 단계 | 엔드포인트 | 처리 |
|----|----------|----|
| 신고 접수 | `POST /api/v1/reports` | `ReportService.createReport()` (Admin Backend로 전달) |
| 신고 상태 확인 | `GET /api/v1/reports/check?targetType=&targetId=` | 처리 대기 중 여부 |
| **WARNING (PR3)** | (Admin → MVP) `POST /api/internal/users/{userId}/warn-from-report` | 누적 경고 +1, 임계치 도달 시 자동 USER_SUSPENDED |
| **USER_SUSPENDED (PR2)** | (Admin → MVP) `POST /api/internal/users/{userId}/suspend-from-report` | 30일 정지, 누적 3회면 영구강퇴 |
| **GUILD_BANNED (PR1c)** | (Admin → MVP) `POST /api/internal/guilds/{guildId}/ban-from-report` | 길드 차단 처리 |

처리 결과는 `notification-service`의 Redis Streams (`AppPushMessageProducer`)를 통해 사용자에게 푸시 + in-app 알림 발송.

### Internal API (Admin Backend ↔ MVP 연동)

`/api/internal/**` 경로는 `SecurityConfig`에서 `permitAll`이지만 **VPC 내부 접근만 허용**. Admin Backend가 MVP의 도메인 데이터를 읽거나 액션을 실행할 때 사용:

| 도메인 | 베이스 경로 | 용도 |
|------|----------|----|
| user | `/api/internal/users` | 유저 검색/조회/통계, blacklist, 신고 처리 (suspend/warn) |
| user | `/api/internal/daily-mvp-exclusions` | MVP 제외 명단 관리 |
| user | `/api/internal/terms` | 약관 관리 |
| guild | `/api/internal/guilds` | 길드 검색, 통계, 활성화 토글, **신고 처리 (ban-from-report)** |
| guild | `/api/internal/guilds/{guildId}` | 길드 게시글 어드민 |
| mission | `/api/internal/missions`, `/api/internal/mission-templates`, `/api/internal/mission-participants`, `/api/internal/mission-comments` | 미션 어드민 |
| feed | `/api/internal/feeds`, `/api/internal/feed-comments` | 피드 어드민 |
| meta | `/api/internal/{user,guild}-level-configs`, `/api/internal/attendance-reward-configs`, `/api/internal/mission-categories`, `/api/internal/profanity-words` | 메타 설정 |
| gamification | `/api/internal/{achievements,achievement-categories,titles,title-grants,events,seasons,check-logic-types,experience-history,mvp-history}` | 게임화 어드민 |
| gamification | `/api/internal/seasons/{id}/rank-rewards` | 시즌 순위 보상 어드민 |

---
> Source: [Level-Up-Together/product-service](https://github.com/Level-Up-Together/product-service) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
