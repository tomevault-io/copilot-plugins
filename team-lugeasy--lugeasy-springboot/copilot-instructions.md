## lugeasy-springboot

> - **프로젝트명**: Lugeasy (러기지 관리 서비스)

# Lugeasy SpringBoot 프로젝트 커서 규칙

## 프로젝트 개요
- **프로젝트명**: Lugeasy (러기지 관리 서비스)
- **기술 스택**: Spring Boot 3.2.2, Java 21, MariaDB, Redis, JWT
- **아키텍처**: 계층형 아키텍처 (Controller → Service → Repository)
- **패키지 구조**: 도메인별 패키지 분리 (user, auth, review, match, common)

## 코딩 컨벤션

### 1. 패키지 구조
```
com.jimjim.lugeasy/
├── common/           # 공통 모듈
│   ├── config/       # 설정 클래스 (SecurityConfig, SwaggerConfig, RedisConfig)
│   ├── entity/       # 공통 엔티티 (BaseEntity)
│   ├── error/        # 에러 처리 (ControllerAdvice, ErrorCode, RestApiException)
│   ├── response/     # 공통 응답 형식 (BaseResponse)
│   └── security/     # 보안 관련 (JWT, AuthenticationMember, CustomUserDetails)
├── {domain}/         # 도메인별 패키지 (user, auth, review, match)
│   ├── application/  # 애플리케이션 서비스 (Facade 패턴)
│   │   └── v1/       # API 버전별 분리
│   │       ├── dto/  # Request/Response DTO
│   │       ├── {Domain}Facade.java      # Facade 인터페이스
│   │       └── {Domain}FacadeImpl.java  # Facade 구현체
│   ├── domain/       # 도메인 엔티티 (Entity, Enum, Value Object)
│   ├── errorCode/    # 도메인별 에러 코드 (DomainErrorCode)
│   ├── infrastructure/ # 리포지토리 (Repository 인터페이스)
│   ├── representation/ # 컨트롤러 (REST API 엔드포인트)
│   │   └── v1/       # API 버전별 분리
│   │       ├── dto/  # 컨트롤러 전용 DTO
│   │       └── {Domain}Controller.java
│   └── service/      # 도메인 서비스 (비즈니스 로직)
│       ├── {Domain}Service.java        # 서비스 인터페이스
│       └── impl/
│           └── {Domain}ServiceImpl.java # 서비스 구현체
```

#### 패키지 구조 설명:
- **application/**: Facade 패턴으로 여러 도메인을 조합하는 애플리케이션 서비스
- **domain/**: 순수한 도메인 로직과 엔티티
- **infrastructure/**: 데이터 접근 계층 (Repository)
- **representation/**: 외부 인터페이스 (REST API)
- **service/**: 도메인별 비즈니스 로직
- **errorCode/**: 도메인별 예외 처리

#### 실제 도메인별 예시:
```
com.jimjim.lugeasy.review/
├── application/v1/
│   ├── dto/
│   │   ├── ReviewRequestDTO.java
│   │   └── ReviewResponseDTO.java
│   ├── ReviewFacade.java
│   └── ReviewFacadeImpl.java
├── domain/
│   └── Review.java
├── errorCode/
│   └── ReviewErrorCode.java
├── infrastructure/
│   └── ReviewRepository.java
├── representation/v1/
│   ├── dto/
│   │   └── ReviewControllerDTO.java
│   └── ReviewController.java
└── service/
    ├── ReviewService.java
    └── impl/
        └── ReviewServiceImpl.java
```

#### 계층 간 의존성 규칙:
1. **Controller** → **Facade** → **Service** → **Repository**
2. **Facade**는 여러 도메인의 **Service**를 조합
3. **Service**는 단일 도메인의 비즈니스 로직만 처리
4. **Repository**는 데이터 접근만 담당

### 2. 네이밍 컨벤션
- **클래스명**: PascalCase (예: `HostController`, `MemberService`)
- **메서드명**: camelCase (예: `getAuthenticatedHostList()`, `updateMember()`)
- **변수명**: camelCase (예: `hostId`, `memberRepository`)
- **상수명**: UPPER_SNAKE_CASE (예: `JWT_SECRET`)
- **패키지명**: 소문자 (예: `com.jimjim.lugeasy.user.domain`)

### 3. 어노테이션 사용 규칙
```java
// 엔티티
@Entity
@Getter
@Builder
@NoArgsConstructor
@AllArgsConstructor
public class Member extends BaseEntity {
    // ...
}

// 서비스
@Service
@RequiredArgsConstructor
@Transactional(readOnly = true) // 조회용
@Transactional // 수정용
public class HostServiceImpl implements HostService {
    // ...
}

// 컨트롤러
@RestController
@RequestMapping("/api/v1/hosts")
@RequiredArgsConstructor
@Tag(name = "Host", description = "호스트 관련 API")
public class HostController {
    // ...
}
```

### 4. 에러 처리 패턴
```java
// 에러 코드 정의
@Getter
@AllArgsConstructor
public enum MemberErrorCode implements ErrorCodeInterface {
    MEMBER_NOT_FOUND("MEMBER001", "Member가 존재하지 않습니다.", HttpStatus.NOT_FOUND);
    
    @Override
    public ErrorCode getErrorCode() {
        return ErrorCode.builder()
                .code(code)
                .message(message)
                .httpStatus(httpStatus)
                .build();
    }
}

// 예외 발생
throw new RestApiException(MemberErrorCode.MEMBER_NOT_FOUND);
```

### 5. API 응답 형식
```java
// 성공 응답
return BaseResponse.onSuccess(result);

// 실패 응답 (ControllerAdvice에서 자동 처리)
throw new RestApiException(ErrorCode);
```

### 6. JPA 엔티티 규칙
```java
@Entity
@Getter
@Builder
@NoArgsConstructor
@AllArgsConstructor
public class Entity extends BaseEntity {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @Column(nullable = false)
    private String name;
    
    @OneToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "member_id", nullable = false)
    private Member member;
    
    @OneToMany(mappedBy = "host", cascade = CascadeType.ALL, orphanRemoval = true)
    private List<HostTime> availableTimes = new ArrayList<>();
}
```

### 7. DTO 패턴
```java
@Getter
@Builder
@NoArgsConstructor
@AllArgsConstructor
@Schema(description = "호스트 상세 정보 DTO")
public class HostDetailResponseDTO {
    @Schema(description = "호스트 ID")
    private Long id;
    
    @Schema(description = "호스트 이름")
    private String name;
}
```

### 8. Swagger 문서화
```java
@Operation(summary = "호스트 목록 조회", description = "인증된 호스트들의 목록을 조회합니다.")
@ApiResponses(value = {
    @ApiResponse(responseCode = "200", description = "호스트 목록 조회 성공",
        content = @Content(mediaType = "application/json",
            examples = @ExampleObject(
                name = "호스트 목록 조회 성공 응답",
                value = """
                {
                  "timestamp": "2025-08-31T09:45:40.426159645Z",
                  "code": "COMMON200",
                  "message": "요청에 성공하였습니다.",
                  "result": [...]
                }
                """
            )
        )
    )
})
```

### 9. 보안 설정
- JWT 기반 인증 사용
- `@AuthenticationMember` 어노테이션으로 현재 사용자 정보 주입
- SecurityConfig에서 경로별 권한 설정

### 10. 설정 파일 규칙
- `application.yml`: 공통 설정
- `application-{profile}.yml`: 프로필별 설정
- `config/` 디렉토리에서 설정 파일 관리

### 11. 트랜잭션 규칙
- 조회 메서드: `@Transactional(readOnly = true)`
- 수정 메서드: `@Transactional`
- 서비스 클래스 레벨에서 트랜잭션 설정

### 12. Lombok 사용 규칙
- `@RequiredArgsConstructor`: final 필드용 생성자
- `@Getter`: getter 메서드 자동 생성
- `@Builder`: 빌더 패턴 자동 생성
- `@NoArgsConstructor`, `@AllArgsConstructor`: 생성자 자동 생성

### 13. 예외 처리 규칙
- 비즈니스 로직 예외: `RestApiException` 사용
- 시스템 예외: `RuntimeException` 사용 (ControllerAdvice에서 처리)
- 에러 코드는 도메인별로 분리하여 관리

### 14. Facade 패턴 규칙
```java
// Facade 인터페이스
public interface ReviewFacade {
    /**
     * 호스트별 리뷰 목록을 조회합니다.
     * 
     * @param hostId 호스트 ID
     * @return 호스트 리뷰 목록 정보
     */
    ReviewResponseDTO.HostReviewList getHostReviews(Long hostId);
}

// Facade 구현체
@Service
@RequiredArgsConstructor
@Transactional(readOnly = true) // 조회용 Facade
@Transactional // 수정용 Facade
public class ReviewFacadeImpl implements ReviewFacade {
    
    // 여러 도메인의 서비스를 조합
    private final ReviewService reviewService;  // Review 도메인
    private final HostService hostService;      // User 도메인
    
    @Override
    @Transactional
    public ReviewResponseDTO.HostReviewList getHostReviews(Long hostId) {
        // 여러 도메인의 서비스를 조합하여 복잡한 비즈니스 로직 처리
        Host host = hostService.getHostById(hostId);  // User 도메인
        List<Review> reviews = reviewService.getReviewsByHostId(hostId);  // Review 도메인
        
        // DTO 변환 및 조합
        return ReviewResponseDTO.HostReviewList.builder()
                .hostId(host.getId())
                .hostName(host.getMember().getName())
                .reviews(reviewDetails)
                .build();
    }
}
```

#### Facade 패턴 사용 시기:
- 여러 도메인의 서비스를 조합해야 할 때
- 복잡한 DTO 변환이 필요할 때
- 트랜잭션 경계를 명확히 하고 싶을 때
- 컨트롤러에서 여러 서비스를 직접 호출하는 것을 방지하고 싶을 때

### 15. 리포지토리 규칙
```java
@Repository
public interface HostRepository extends JpaRepository<Host, Long> {
    // 커스텀 쿼리 메서드
    @Query("SELECT h FROM Host h JOIN FETCH h.member WHERE h.isAuthentication = true")
    List<Host> findAuthenticatedHostsWithMember();
}
```

### 16. 테스트 데이터
- `insert_test_data.sql`에 테스트 데이터 정의
- 실제 비즈니스 시나리오를 반영한 테스트 데이터 작성

## 주의사항
1. 모든 API는 `BaseResponse`로 래핑하여 응답
2. 에러 처리는 `ControllerAdvice`에서 중앙 집중식으로 처리
3. JPA 엔티티는 `BaseEntity`를 상속받아 공통 필드 관리
4. Swagger 문서화는 모든 API에 필수
5. 보안이 필요한 API는 JWT 인증 필수
6. 트랜잭션은 서비스 레이어에서 관리
7. 도메인별로 패키지를 분리하여 관심사 분리
8. **Facade 패턴**: 여러 도메인의 서비스를 조합하는 복잡한 비즈니스 로직은 Facade에서 처리

### 17. Controller 계층 규칙
```java
// Controller는 단순히 Facade 메서드 호출만 담당
@RestController
@RequestMapping("/api/v1/example")
@RequiredArgsConstructor
public class ExampleController {
    
    private final ExampleFacade exampleFacade;
    
    @PostMapping
    public BaseResponse<ExampleResponse> createExample(
            @RequestBody ExampleRequest request,
            @AuthenticationMember Long memberId) {
        
        // Controller에서는 단순히 Facade 호출만
        return BaseResponse.onSuccess(exampleFacade.createExample(request, memberId));
    }
}
```

#### Controller 계층 금지사항:
- ❌ 비즈니스 로직 구현
- ❌ 데이터 변환 로직
- ❌ 권한 확인 로직
- ❌ 복잡한 조건문이나 반복문
- ❌ 여러 서비스 직접 호출

#### Controller 계층 허용사항:
- ✅ Facade 메서드 호출
- ✅ Request/Response 매핑
- ✅ Swagger 어노테이션
- ✅ 인증 정보 주입 (@AuthenticationMember)

### 18. @AuthenticationMember 사용 규칙
```java
// ✅ 올바른 사용법: Member 객체를 받아서 ID 추출
@GetMapping
public BaseResponse<ExampleResponse> getExample(
        @AuthenticationMember Member member) {
    
    return BaseResponse.onSuccess(exampleFacade.getExample(member.getId()));
}

// ❌ 잘못된 사용법: Long memberId 직접 사용
@GetMapping
public BaseResponse<ExampleResponse> getExample(
        @AuthenticationMember Long memberId) {  // null 반환됨
    
    return BaseResponse.onSuccess(exampleFacade.getExample(memberId));
}
```

#### @AuthenticationMember 규칙:
- **반드시 `Member` 객체로 받기**: `@AuthenticationMember Member member`
- **ID 추출**: `member.getId()`로 memberId 사용
- **null 체크 불필요**: 인증된 사용자만 접근 가능한 API에서 사용
- **일관성 유지**: 모든 Controller에서 동일한 패턴 사용

## 코드 작성 시 준수사항
- 기존 코드 스타일과 일관성 유지
- 한국어 주석 사용 (비즈니스 로직 설명)
- 영어 네이밍 사용 (클래스, 메서드, 변수명)
- 적절한 예외 처리 및 로깅
- 성능을 고려한 JPA 쿼리 작성
- 보안 취약점 방지

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Team-Lugeasy) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
