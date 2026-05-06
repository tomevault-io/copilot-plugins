## umc-product-server

> 이 문서는 GitHub Copilot이 코드 리뷰 시 참고해야 할 프로젝트별 가이드라인입니다.

# UMC Challenger - Copilot Code Review Instructions

## Overview

이 문서는 GitHub Copilot이 코드 리뷰 시 참고해야 할 프로젝트별 가이드라인입니다.  
헥사고날 아키텍처를 따르는 Spring Boot 프로젝트입니다.

---

## Architecture Rules

### Package Structure Validation

```
✅ 올바른 구조:
{domain}/
├── domain/           # Entity, VO, Enum, Domain Service
├── application/
│   ├── port/in/     # UseCase interfaces
│   ├── port/out/    # Repository interfaces
│   └── service/     # UseCase implementations
└── adapter/
    ├── in/          # Controllers, Schedulers
    └── out/         # Persistence, External APIs

❌ 잘못된 구조:
- domain/ 내에 Repository 인터페이스
- adapter/ 내에 비즈니스 로직
- application/service/에 인터페이스 정의
```

### Dependency Direction

```
✅ 허용된 의존 방향:
adapter/in → application/service → application/port
adapter/out → application/port/out
application/service → domain

❌ 금지된 의존:
domain → application (역방향)
domain → adapter (역방향)
application/port → application/service (역방향)
adapter/in → adapter/out (수평 의존)
```

---

## Code Review Checklist

### 1. Entity Review

```java
// ✅ GOOD
@Entity
@Getter
@NoArgsConstructor(access = AccessLevel.PROTECTED)
public class Challenger extends BaseEntity {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private Long userId;  // ID 참조만

    @Builder
    private Challenger(...) {
    }

    // 도메인 로직
    public void graduate() {
        validateActive();
        this.status = ChallengerStatus.GRADUATED;
    }
}

// ❌ BAD
@Entity
@Getter
@Setter  // Setter 금지
@NoArgsConstructor  // access level 누락
public class Challenger {
    @ManyToOne
    private Member member;  // 다른 도메인 직접 참조 금지

    // 도메인 로직 없이 getter/setter만
}
```

**체크포인트:**

- [ ] `@Setter` 사용하지 않음
- [ ] `@NoArgsConstructor(access = AccessLevel.PROTECTED)` 사용
- [ ] 다른 도메인 Entity 직접 참조하지 않음 (ID만 저장)
- [ ] 상태 변경은 도메인 메서드로만
- [ ] Builder는 private 또는 package-private
- [ ] `BaseEntity` 상속 (created_at, updated_at)

### 2. UseCase (Port In) Review

```java
// ✅ GOOD
public interface RegisterChallengerUseCase {
    Long register(RegisterChallengerCommand command);
}

public record RegisterChallengerCommand(
        Long userId,
        Long gisuId,
        ChallengerPart part
) {
    // Validation in constructor if needed
    public RegisterChallengerCommand {
        Objects.requireNonNull(userId, "userId must not be null");
    }
}

// ❌ BAD
public interface ChallengerUseCase {  // 너무 포괄적인 이름
    void register(Long userId, Long gisuId, String part);  // primitive 타입 나열

    Challenger getById(Long id);  // Entity 직접 반환
}
```

**체크포인트:**

- [ ] 단일 책임: 하나의 UseCase는 하나의 기능만
- [ ] Command/Query 분리 (CQRS)
- [ ] Command 객체는 record 사용 권장
- [ ] Entity 대신 Info/DTO 반환
- [ ] 네이밍: `{동작}{도메인}UseCase`

### 3. Port (Port Out) Review

```java
// ✅ GOOD
public interface LoadChallengerPort {
    Optional<Challenger> findById(Long id);

    boolean existsByUserIdAndGisuId(Long userId, Long gisuId);
}

public interface SaveChallengerPort {
    Challenger save(Challenger challenger);
}

// ❌ BAD
public interface ChallengerPort {  // Load/Save 분리 안됨
    Challenger findById(Long id);  // Optional 미사용

    void save(Challenger challenger);  // 반환값 없음

    List<ChallengerResponse> findAllWithUserInfo();  // Response DTO 반환
}
```

**체크포인트:**

- [ ] Load/Save Port 분리 (ISP 원칙)
- [ ] `Optional` 적절히 사용
- [ ] Response DTO가 아닌 Entity/Domain 객체 반환
- [ ] 메서드명은 기술 중립적으로

### 4. Service Review

```java
// ✅ GOOD
@Service
@RequiredArgsConstructor
@Transactional(readOnly = true)
public class ChallengerQueryService implements GetChallengerUseCase {

    private final LoadChallengerPort loadChallengerPort;

    @Override
    public ChallengerInfo getById(Long challengerId) {
        Challenger challenger = loadChallengerPort.findById(challengerId)
                .orElseThrow(() -> new BusinessException(ErrorCode.CHALLENGER_NOT_FOUND));
        return ChallengerInfo.from(challenger);
    }
}

@Service
@RequiredArgsConstructor
@Transactional
public class ChallengerCommandService implements RegisterChallengerUseCase {
    // Command는 @Transactional (readOnly = false)
}

// ❌ BAD
@Service
public class ChallengerService {  // Command/Query 미분리
    @Autowired  // 필드 주입
    private ChallengerRepository repository;  // Port 대신 Repository 직접 사용

    public ChallengerResponse register(...) {  // Response DTO 반환
        // 검증 로직 없이 바로 저장
        return new ChallengerResponse(repository.save(...));
    }
}
```

**체크포인트:**

- [ ] 생성자 주입 (`@RequiredArgsConstructor`)
- [ ] Command/Query Service 분리
- [ ] Query Service는 `@Transactional(readOnly = true)`
- [ ] Port 인터페이스에 의존 (Repository 직접 사용 X)
- [ ] 적절한 예외 처리
- [ ] Response DTO 대신 Info 객체 반환

### 5. Adapter Review

```java
// ✅ GOOD - Persistence Adapter
@Component
@RequiredArgsConstructor
public class ChallengerPersistenceAdapter
        implements LoadChallengerPort, SaveChallengerPort {

    private final ChallengerRepository repository;

    @Override
    public Optional<Challenger> findById(Long id) {
        return repository.findById(id);
    }
}

// ✅ GOOD - Controller (In Adapter)
@RestController
@RequestMapping("/api/v1/challengers")
@RequiredArgsConstructor
public class ChallengerController {

    private final RegisterChallengerUseCase registerUseCase;

    @PostMapping
    public ApiResponse<Long> register(
            @AuthenticationPrincipal SecurityUser member,
            @Valid @RequestBody RegisterChallengerRequest request) {

        Long id = registerUseCase.register(request.toCommand(member.getUserId()));
        return ApiResponse.success(id);
    }
}

// ❌ BAD
@RestController
public class ChallengerController {
    private final ChallengerRepository repository;  // Repository 직접 사용

    @PostMapping("/challenger")  // 복수형 미사용
    public Challenger register(@RequestBody Map<String, Object> request) {  // Map 사용
        // Controller에서 비즈니스 로직 처리
        Challenger challenger = new Challenger();
        challenger.setUserId((Long) request.get("userId"));
        return repository.save(challenger);
    }
}
```

**체크포인트:**

- [ ] Controller는 UseCase에만 의존
- [ ] Controller에 비즈니스 로직 없음
- [ ] Request/Response DTO 사용 (Map, Entity 직접 사용 X)
- [ ] `@Valid` 어노테이션으로 입력 검증
- [ ] REST 네이밍 규칙 준수 (복수형, 케밥 케이스)
- [ ] Persistence Adapter는 Port 인터페이스 구현

### 6. DTO Review

```java
// ✅ GOOD - Request
public record RegisterChallengerRequest(
                @NotNull Long gisuId,
                @NotNull ChallengerPart part
        ) {
    public RegisterChallengerCommand toCommand(Long userId) {
        return new RegisterChallengerCommand(userId, gisuId, part);
    }
}

// ✅ GOOD - Response
public record ChallengerResponse(
        Long id,
        String userName,
        String part,
        String status
) {
    public static ChallengerResponse from(ChallengerInfo info) {
        return new ChallengerResponse(
                info.id(),
                info.userName(),
                info.part().name(),
                info.status().name()
        );
    }
}

// ❌ BAD
@Data  // Lombok @Data 사용
public class ChallengerRequest {
    private Long userId;  // 인증 정보를 Request에 포함
    // validation 어노테이션 없음
}
```

**체크포인트:**

- [ ] record 사용 권장 (불변성)
- [ ] Request에 인증 정보 포함하지 않음
- [ ] 적절한 Validation 어노테이션
- [ ] `toCommand()`, `from()` 변환 메서드
- [ ] Enum은 name()으로 문자열 변환

---

## Naming Convention Validation

### File Names

| Type       | Pattern                                | Example                             |
|------------|----------------------------------------|-------------------------------------|
| Entity     | `{Domain}.java`                        | `Challenger.java`                   |
| Enum       | `{Domain}{Type}.java`                  | `ChallengerStatus.java`             |
| UseCase    | `{Action}{Domain}UseCase.java`         | `RegisterChallengerUseCase.java`    |
| Port       | `{Action}{Domain}Port.java`            | `LoadChallengerPort.java`           |
| Service    | `{Domain}{Command\|Query}Service.java` | `ChallengerCommandService.java`     |
| Controller | `{Domain}Controller.java`              | `ChallengerController.java`         |
| Request    | `{Action}{Domain}Request.java`         | `RegisterChallengerRequest.java`    |
| Response   | `{Domain}{Purpose}Response.java`       | `ChallengerListResponse.java`       |
| Adapter    | `{Domain}{Tech}Adapter.java`           | `ChallengerPersistenceAdapter.java` |

### Method Names

```java
// UseCase methods
register(),create(),

update(),delete()  // Command

getById(),getAll(),

search(),find()     // Query

// Port methods
save(),delete()                          // Save Port

findById(),findAll(),existsBy...()      // Load Port

// Controller endpoints
POST   /api/v1/{domains}           // 생성
GET    /api/v1/{domains}           // 목록
GET    /api/v1/{domains}/{id}      // 단건
PATCH  /api/v1/{domains}/{id}      // 수정
DELETE /api/v1/{domains}/{id}      // 삭제
```

---

## Common Anti-Patterns to Flag

### 1. God Service

```java
// ❌ 하나의 서비스가 너무 많은 책임
@Service
public class ChallengerService {
    public void register() {
    }

    public void assignRole() {
    }

    public void addRewardPenalty() {
    }

    public void graduate() {
    }

    public List<ChallengerResponse> search() {
    }

    public void sendNotification() {
    }  // 다른 도메인 책임
}
```

**권장:** 기능별로 서비스 분리

### 2. Anemic Domain Model

```java
// ❌ Entity에 로직 없이 getter/setter만
@Entity
@Data
public class Challenger {
    private ChallengerStatus status;
}

// Service에서 직접 상태 변경
challenger.

setStatus(ChallengerStatus.GRADUATED);
```

**권장:** Entity에 도메인 로직 포함

```java
challenger.graduate();  // Entity 메서드 호출
```

### 3. Leaking Domain

```java
// ❌ Controller에서 Entity 직접 반환
@GetMapping("/{id}")
public Challenger getById(@PathVariable Long id) {
    return repository.findById(id).orElseThrow();
}
```

**권장:** Response DTO로 변환 후 반환

### 4. Cross-Domain Direct Access

```java
// ❌ 다른 도메인 Repository 직접 사용
@Service
public class ChallengerCommandService {
    private final UserRepository userRepository;  // 다른 도메인

    public void register(...) {
        User member = userRepository.findById(userId);  // 직접 접근
    }
}
```

**권장:** 다른 도메인의 Query UseCase 사용

```java
private final GetUserInfoUseCase getUserInfoUseCase;

public void register(...) {
    UserInfo member = getUserInfoUseCase.getById(userId);
}
```

### 5. Missing Transaction Boundary

```java
// ❌ Transaction 어노테이션 누락
@Service
public class ChallengerCommandService {
    public void register(...) {
        // 여러 저장 작업이 원자적으로 처리되지 않음
    }
}
```

**권장:** Command Service에 `@Transactional` 필수

---

## Security Review Points

### Authentication

```java
// ✅ 인증된 사용자 정보는 @AuthenticationPrincipal로
@PostMapping
public ApiResponse<Long> register(
        @AuthenticationPrincipal SecurityUser member,
        @RequestBody RegisterRequest request) {
    // member.getUserId() 사용
}

// ❌ Request Body에서 userId 받지 않음
public record RegisterRequest(
        Long userId,  // 보안 취약점
    ...
)
```

### Authorization

```java
// ✅ 권한 검증은 Service 레이어에서
public void updateNotice(Long noticeId, UpdateCommand command, Long requesterId) {
    Notice notice = loadNoticePort.findById(noticeId);

    // 작성자 또는 상위 권한 확인
    if (!notice.isAuthor(requesterId) && !hasManagePermission(requesterId)) {
        throw new BusinessException(ErrorCode.FORBIDDEN);
    }
}
```

---

## Performance Review Points

### N+1 Query

```java
// ❌ N+1 발생 가능
List<Challenger> challengers = repository.findAll();
for(
Challenger c :challengers){
User member = userRepository.findById(c.getUserId());  // N번 쿼리
}

// ✅ Fetch Join 또는 별도 쿼리
@Query("SELECT c FROM Challenger c WHERE c.gisuId = :gisuId")
List<Challenger> findByGisuId(@Param("gisuId") Long gisuId);

// 이후 필요한 User 정보는 IN 쿼리로 한번에
List<Long> userIds = challengers.stream().map(Challenger::getUserId).toList();
Map<Long, UserInfo> userMap = getUserInfoUseCase.getByIds(userIds);
```

### Pagination

```java
// ✅ 페이징 적용
Page<Challenger> findByGisuId(Long gisuId, Pageable pageable);

// ❌ 전체 조회
List<Challenger> findByGisuId(Long gisuId);
```

---

## Test Code Review

### Test Naming

```java
// ✅ 한글 메서드명으로 명확하게
@Test
void 챌린저_등록_성공() {
}

@Test
void 존재하지_않는_사용자면_USER_NOT_FOUND_예외() {
}

@Test
void 이미_등록된_챌린저면_중복_예외() {
}

// ❌ 불명확한 테스트명
@Test
void test1() {
}

@Test
void registerTest() {
}
```

### Test Structure

```java

@Test
void 챌린저_등록_성공() {
    // given - 테스트 데이터 준비
    var command = new RegisterChallengerCommand(1L, 1L, ChallengerPart.WEB);
    given(getUserInfoUseCase.existsById(1L)).willReturn(true);

    // when - 테스트 대상 실행
    Long result = sut.register(command);

    // then - 결과 검증
    assertThat(result).isNotNull();
    then(saveChallengerPort).should().save(any());
}
```

---

## Review Comment Templates

### Architecture Violation

```
🏗️ **Architecture Violation**
{layer}에서 {dependency}를 직접 참조하고 있습니다.
헥사고날 아키텍처에서는 {correct_approach}를 통해 접근해야 합니다.

예시:
- Controller → UseCase → Port (O)
- Controller → Repository (X)
```

### Naming Convention

```
📝 **Naming Convention**
현재: `{current_name}`
권장: `{suggested_name}`

네이밍 규칙: {rule_description}
```

### Missing Validation

```
⚠️ **Missing Validation**
입력값 검증이 누락되었습니다.
- `{field}` 필드에 `@{annotation}` 추가 필요
- null 체크 또는 비즈니스 규칙 검증 필요
```

### Security Issue

````
🔒 **Security Issue**
{description}

권장 수정:
```java
{code_suggestion}
````

```

### Performance Concern

```

⚡ **Performance Concern**
{description}

발생 가능한 문제: {issue}
권장 해결책: {solution}

```

---

## Auto-Review Priorities

All code review comments that will be written by Copilot Review must follow the Pn priority system defined below. Each
comment must begin with a Pn label (e.g., P3).

Pn Level Selection Criteria
P1: Only use for critical issues such as severe bugs, potential service failures, security vulnerabilities, or data
loss/corruption. Use with extreme caution.
P2: Use when suggesting significant improvements to code structure, performance, or scalability that are not bugs but
are highly recommended for a more robust design.
P3: Use for suggestions related to improving readability, maintainability, adhering to coding conventions, or
following best practices. This should be the default for general improvement suggestions.
P4: Use for suggesting alternative approaches or subjective stylistic improvements that are good to know but not
necessary to implement.
P5: Use for minor comments such as fixing typos, asking questions, or giving compliments that require little to no
code change.
Commenting Guidelines
Always provide a clear and concise explanation for your suggestion along with the Pn label.
When suggesting code changes, always use markdown code block format.
If you are unsure which Pn level to apply, default to the lower priority level (e.g., choose P3 if you are debating
between P2 and P3).

By following these instructions, you will help maintain the consistency and quality of the `UMC Product Team Backend`
codebase.

Lastly, translate your review comments in Korean (including the Pull Request Overview) before submitting.
```

---
> Source: [UMC-PRODUCT/umc-product-server](https://github.com/UMC-PRODUCT/umc-product-server) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-02 -->
