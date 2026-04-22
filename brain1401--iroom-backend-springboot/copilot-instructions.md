## iroom-backend-springboot

> REST API 설계 및 응답 표준화 규칙


# API 설계 및 응답 표준화

## 🎯 API 설계 원칙

### RESTful API 설계
- **리소스 중심** URL 설계
- **HTTP 메서드** 의미에 맞는 사용
- **일관된 응답 형식** (ApiResponse 래퍼)
- **적절한 상태 코드** 사용

### API 문서화
- **OpenAPI 3.0** (SpringDoc) 활용
- **@Tag**, **@Operation** 상세 작성
- **@Schema** 어노테이션으로 DTO 문서화

## 📋 표준 응답 형식 (ApiResponse)

### ✅ 기본 ApiResponse 구조
```java
/**
 * API 표준 응답 래퍼
 */
public record ApiResponse<T>(
    @Schema(description = "처리 결과", example = "SUCCESS") 
    String result,
    
    @Schema(description = "응답 메시지", example = "요청이 성공적으로 처리되었습니다") 
    String message,
    
    @Schema(description = "응답 데이터") 
    T data
) {
    public ApiResponse {
        // 불변성: result, message는 null이 될 수 없음
        if (result == null || result.isBlank()) 
            throw new IllegalArgumentException("result는 필수입니다");
        if (message == null) 
            throw new IllegalArgumentException("message는 필수입니다");
    }
    
    /**
     * 성공 응답 생성 (데이터 포함)
     */
    public static <T> ApiResponse<T> success(String message, T data) {
        return new ApiResponse<>("SUCCESS", message, data);
    }
    
    /**
     * 성공 응답 생성 (데이터 없음)
     */
    public static ApiResponse<Void> success(String message) {
        return new ApiResponse<>("SUCCESS", message, null);
    }
    
    /**
     * 오류 응답 생성
     */
    public static ApiResponse<Void> error(String message) {
        return new ApiResponse<>("ERROR", message, null);
    }
}
```

### ✅ Controller에서 ApiResponse 활용
```java
/**
 * 사용자 관리 컨트롤러
 */
@RestController
@RequestMapping("/users")
@RequiredArgsConstructor
@Tag(name = "사용자 API", description = "학생 로그인 및 정보 관리 API")
@Validated
public class UserController {
    
    private final UserService userService;
    
    /**
     * 사용자 로그인
     */
    @PostMapping("/login")
    @Operation(
        summary = "사용자 로그인",
        description = "학생이 이름과 전화번호로 로그인합니다",
        responses = {
            @ApiResponse(responseCode = "200", description = "로그인 성공"),
            @ApiResponse(responseCode = "400", description = "입력 데이터 검증 실패"),
            @ApiResponse(responseCode = "404", description = "존재하지 않는 사용자")
        }
    )
    public ApiResponse<UserLoginResponse> login(
            @Parameter(description = "로그인 요청 정보") 
            @Valid @RequestBody UserLoginRequest request) {
        
        UserLoginResponse response = userService.login(request);
        return ApiResponse.success("로그인에 성공했습니다", response);
    }
    
    /**
     * 사용자 정보 조회
     */
    @GetMapping("/{id}")
    @Operation(summary = "사용자 정보 조회", description = "사용자 ID로 상세 정보를 조회합니다")
    public ApiResponse<UserDto> getUser(
            @Parameter(description = "사용자 ID", example = "1") 
            @PathVariable Long id) {
        
        UserDto user = userService.findById(id);
        return ApiResponse.success("사용자 정보를 조회했습니다", user);
    }
    
    /**
     * 사용자 목록 조회 (페이징)
     */
    @GetMapping
    @Operation(summary = "사용자 목록 조회", description = "페이징을 지원하는 사용자 목록을 조회합니다")
    public ApiResponse<Page<UserDto>> getUsers(
            @Parameter(description = "페이지 번호 (0부터 시작)", example = "0")
            @RequestParam(defaultValue = "0") int page,
            
            @Parameter(description = "페이지 크기", example = "20")
            @RequestParam(defaultValue = "20") int size,
            
            @Parameter(description = "정렬 기준", example = "name")
            @RequestParam(defaultValue = "id") String sort,
            
            @Parameter(description = "정렬 방향", example = "asc")
            @RequestParam(defaultValue = "asc") String direction) {
        
        Sort sortOrder = Sort.by(Sort.Direction.fromString(direction), sort);
        Pageable pageable = PageRequest.of(page, size, sortOrder);
        
        Page<UserDto> users = userService.findAll(pageable);
        return ApiResponse.success("사용자 목록을 조회했습니다", users);
    }
}
```

## 🔢 HTTP 상태 코드 활용

### 성공 상태 코드
| 코드               | 설명            | 사용 시점            | 예시              |
| ------------------ | --------------- | -------------------- | ----------------- |
| **200 OK**         | 요청 성공       | GET, PUT, PATCH 성공 | 조회, 수정 완료   |
| **201 Created**    | 생성 성공       | POST 성공            | 사용자, 시험 생성 |
| **204 No Content** | 성공, 응답 없음 | DELETE 성공          | 삭제 완료         |

### ✅ 상태 코드 적용 예시
```java
@RestController
@RequestMapping("/exams")
@RequiredArgsConstructor
@Tag(name = "시험 관리 API")
public class ExamController {
    
    private final ExamService examService;
    
    /**
     * 시험 생성 - 201 Created
     */
    @PostMapping
    @ResponseStatus(HttpStatus.CREATED)
    @Operation(summary = "시험 생성", description = "새로운 시험을 등록합니다")
    public ApiResponse<ExamCreateResponse> createExam(
            @Valid @RequestBody ExamCreateRequest request) {
        
        ExamCreateResponse exam = examService.createExam(request);
        return ApiResponse.success("시험이 성공적으로 등록되었습니다", exam);
    }
    
    /**
     * 시험 조회 - 200 OK
     */
    @GetMapping("/{id}")
    @Operation(summary = "시험 상세 조회")
    public ApiResponse<ExamDetailResponse> getExam(@PathVariable Long id) {
        ExamDetailResponse exam = examService.getExamDetail(id);
        return ApiResponse.success("시험 정보를 조회했습니다", exam);
    }
    
    /**
     * 시험 수정 - 200 OK
     */
    @PutMapping("/{id}")
    @Operation(summary = "시험 정보 수정")
    public ApiResponse<ExamDetailResponse> updateExam(
            @PathVariable Long id,
            @Valid @RequestBody ExamUpdateRequest request) {
        
        ExamDetailResponse exam = examService.updateExam(id, request);
        return ApiResponse.success("시험 정보가 수정되었습니다", exam);
    }
    
    /**
     * 시험 삭제 - 204 No Content
     */
    @DeleteMapping("/{id}")
    @ResponseStatus(HttpStatus.NO_CONTENT)
    @Operation(summary = "시험 삭제")
    public void deleteExam(@PathVariable Long id) {
        examService.deleteExam(id);
        // 204는 응답 바디가 없으므로 ApiResponse 사용 안함
    }
}
```

## 🚨 에러 응답 처리

### ✅ 전역 예외 처리
```java
/**
 * 전역 예외 처리 핸들러
 */
@RestControllerAdvice
@Slf4j
public class GlobalExceptionHandler {
    
    /**
     * 비즈니스 예외 처리 - 400 Bad Request
     */
    @ExceptionHandler(IllegalArgumentException.class)
    @ResponseStatus(HttpStatus.BAD_REQUEST)
    public ApiResponse<Void> handleIllegalArgumentException(IllegalArgumentException e) {
        log.warn("비즈니스 규칙 위반: {}", e.getMessage());
        return ApiResponse.error(e.getMessage());
    }
    
    /**
     * 리소스 없음 예외 처리 - 404 Not Found
     */
    @ExceptionHandler({UserNotFoundException.class, ExamNotFoundException.class})
    @ResponseStatus(HttpStatus.NOT_FOUND)
    public ApiResponse<Void> handleNotFoundException(RuntimeException e) {
        log.warn("리소스 없음: {}", e.getMessage());
        return ApiResponse.error(e.getMessage());
    }
    
    /**
     * 입력 데이터 검증 실패 - 400 Bad Request
     */
    @ExceptionHandler(MethodArgumentNotValidException.class)
    @ResponseStatus(HttpStatus.BAD_REQUEST)
    public ApiResponse<Map<String, String>> handleValidationException(MethodArgumentNotValidException e) {
        Map<String, String> errors = new HashMap<>();
        e.getBindingResult().getFieldErrors().forEach(error -> 
            errors.put(error.getField(), error.getDefaultMessage()));
        
        log.warn("입력 데이터 검증 실패: {}", errors);
        return ApiResponse.error("입력 데이터 검증에 실패했습니다");
    }
    
    /**
     * 중복 리소스 예외 처리 - 409 Conflict
     */
    @ExceptionHandler(DuplicateResourceException.class)
    @ResponseStatus(HttpStatus.CONFLICT)
    public ApiResponse<Void> handleDuplicateResourceException(DuplicateResourceException e) {
        log.warn("중복 리소스: {}", e.getMessage());
        return ApiResponse.error(e.getMessage());
    }
    
    /**
     * AI 서비스 예외 처리 - 502 Bad Gateway
     */
    @ExceptionHandler(AiRecognitionException.class)
    @ResponseStatus(HttpStatus.BAD_GATEWAY)
    public ApiResponse<Void> handleAiRecognitionException(AiRecognitionException e) {
        log.error("AI 서비스 오류: {}", e.getMessage(), e);
        return ApiResponse.error("AI 인식 서비스에 일시적인 문제가 발생했습니다. 잠시 후 다시 시도해주세요.");
    }
    
    /**
     * 일반적인 서버 오류 - 500 Internal Server Error
     */
    @ExceptionHandler(Exception.class)
    @ResponseStatus(HttpStatus.INTERNAL_SERVER_ERROR)
    public ApiResponse<Void> handleGenericException(Exception e) {
        log.error("예상치 못한 서버 오류: {}", e.getMessage(), e);
        return ApiResponse.error("서버 내부 오류가 발생했습니다. 관리자에게 문의해주세요.");
    }
}
```

## 📄 페이징 및 정렬

### ✅ 표준 페이징 파라미터
```java
/**
 * 페이징 지원 API 예시
 */
@GetMapping("/questions")
@Operation(summary = "문제 목록 조회", description = "페이징과 정렬을 지원하는 문제 목록 조회")
public ApiResponse<Page<QuestionListResponse>> getQuestions(
        @Parameter(description = "학년 필터", example = "1")
        @RequestParam(required = false) Integer grade,
        
        @Parameter(description = "난이도 필터", example = "중")
        @RequestParam(required = false) Question.Difficulty difficulty,
        
        @Parameter(description = "검색 키워드")
        @RequestParam(required = false) String search,
        
        @Parameter(description = "페이지 번호 (0부터 시작)", example = "0")
        @RequestParam(defaultValue = "0") 
        @Min(0) int page,
        
        @Parameter(description = "페이지 크기 (1-100)", example = "20")
        @RequestParam(defaultValue = "20") 
        @Min(1) @Max(100) int size,
        
        @Parameter(description = "정렬 필드", example = "createdAt")
        @RequestParam(defaultValue = "id") String sort,
        
        @Parameter(description = "정렬 방향", example = "desc")
        @RequestParam(defaultValue = "asc") String direction) {
    
    // 정렬 설정
    Sort sortOrder = Sort.by(Sort.Direction.fromString(direction), sort);
    Pageable pageable = PageRequest.of(page, size, sortOrder);
    
    // 검색 조건 설정
    QuestionSearchCondition condition = QuestionSearchCondition.builder()
        .grade(grade)
        .difficulty(difficulty)
        .search(search)
        .build();
    
    Page<QuestionListResponse> questions = questionService.searchQuestions(condition, pageable);
    return ApiResponse.success("문제 목록을 조회했습니다", questions);
}
```

### ✅ 페이징 응답 예시
```json
{
  "result": "SUCCESS",
  "message": "문제 목록을 조회했습니다",
  "data": {
    "content": [
      {
        "id": 1,
        "questionText": "다음 연립부등식을 풀어보시오",
        "difficulty": "중",
        "unitName": "부등식"
      }
    ],
    "pageable": {
      "sort": {"empty": false, "sorted": true, "unsorted": false},
      "pageNumber": 0,
      "pageSize": 20
    },
    "totalElements": 150,
    "totalPages": 8,
    "last": false,
    "first": true,
    "numberOfElements": 20
  }
}
```

## 🔍 API 문서화 패턴

### ✅ 상세한 OpenAPI 문서화
```java
/**
 * 시험 답안 관리 컨트롤러
 */
@RestController
@RequestMapping("/exam-answers")
@RequiredArgsConstructor
@Tag(
    name = "시험 답안 API", 
    description = """
        학생의 시험 답안 제출 및 관리를 위한 API입니다.
        
        주요 기능:
        - AI 이미지 인식을 통한 답안 자동 처리
        - 수동 답안 수정 및 재채점
        - 답안지 전체 촬영 및 일괄 처리
        """
)
public class ExamAnswerController {
    
    private final ExamAnswerService examAnswerService;
    
    /**
     * 답안 생성 (AI 인식 포함)
     */
    @PostMapping
    @ResponseStatus(HttpStatus.CREATED)
    @Operation(
        summary = "답안 생성",
        description = """
            학생이 문제에 대한 답안을 제출합니다.
            업로드된 이미지를 AI가 자동으로 인식하여 텍스트로 변환하고 채점을 수행합니다.
            """,
        responses = {
            @ApiResponse(
                responseCode = "201", 
                description = "답안 생성 성공",
                content = @Content(schema = @Schema(implementation = ExamAnswerResponse.class))
            ),
            @ApiResponse(
                responseCode = "400", 
                description = "입력 데이터 검증 실패",
                content = @Content(schema = @Schema(implementation = ApiResponse.class))
            ),
            @ApiResponse(
                responseCode = "404", 
                description = "시험 제출 또는 문제를 찾을 수 없음",
                content = @Content(schema = @Schema(implementation = ApiResponse.class))
            ),
            @ApiResponse(
                responseCode = "409", 
                description = "이미 답안이 존재함",
                content = @Content(schema = @Schema(implementation = ApiResponse.class))
            )
        }
    )
    public ApiResponse<ExamAnswerResponse> createAnswer(
            @Parameter(
                description = "답안 생성 요청 정보", 
                required = true,
                content = @Content(schema = @Schema(implementation = ExamAnswerCreateRequest.class))
            )
            @Valid @RequestBody ExamAnswerCreateRequest request) {
        
        ExamAnswerResponse response = examAnswerService.createExamAnswer(request);
        return ApiResponse.success("답안이 성공적으로 생성되었습니다", response);
    }
}
```

### ✅ Request/Response DTO 문서화
```java
/**
 * 답안 생성 요청 DTO
 */
@Schema(description = "시험 답안 생성 요청")
public record ExamAnswerCreateRequest(
    @NotNull(message = "시험 제출 ID는 필수입니다")
    @Schema(description = "시험 제출 ID", example = "1", requiredMode = Schema.RequiredMode.REQUIRED)
    Long examSubmissionId,
    
    @NotNull(message = "문제 ID는 필수입니다")
    @Schema(description = "문제 ID", example = "5", requiredMode = Schema.RequiredMode.REQUIRED)
    Long questionId,
    
    @NotBlank(message = "답안 이미지 URL은 필수입니다")
    @URL(message = "올바른 URL 형식이 아닙니다")
    @Schema(description = "답안 이미지 URL", example = "https://example.com/answer-image.jpg", requiredMode = Schema.RequiredMode.REQUIRED)
    String answerImageUrl
) {
    public ExamAnswerCreateRequest {
        Objects.requireNonNull(examSubmissionId, "examSubmissionId는 필수입니다");
        Objects.requireNonNull(questionId, "questionId는 필수입니다");
        Objects.requireNonNull(answerImageUrl, "answerImageUrl는 필수입니다");
    }
}

/**
 * 답안 응답 DTO
 */
@Schema(description = "시험 답안 정보")
public record ExamAnswerResponse(
    @Schema(description = "답안 ID", example = "1")
    Long id,
    
    @Schema(description = "문제 ID", example = "5")
    Long questionId,
    
    @Schema(description = "답안 이미지 URL", example = "https://example.com/answer-image.jpg")
    String answerImageUrl,
    
    @Schema(description = "인식된 답안 텍스트", example = "x = 5")
    String answerText,
    
    @Schema(description = "정답 여부", example = "true")
    Boolean isCorrect,
    
    @Schema(description = "획득 점수", example = "5")
    Integer score,
    
    @Schema(description = "답안 생성 시간", example = "2024-08-17T10:30:00")
    LocalDateTime createdAt
) {
    /**
     * Entity에서 DTO로 변환
     */
    public static ExamAnswerResponse from(ExamAnswer examAnswer) {
        return new ExamAnswerResponse(
            examAnswer.getId(),
            examAnswer.getQuestion().getId(),
            examAnswer.getAnswerImageUrl(),
            examAnswer.getAnswerText(),
            examAnswer.getIsCorrect(),
            examAnswer.getScore(),
            examAnswer.getCreatedAt()
        );
    }
}
```

## 🎯 API 설계 체크리스트

새 API 엔드포인트 작성 시:
- [ ] RESTful URL 설계 (명사 사용, 동사 금지)
- [ ] 적절한 HTTP 메서드 사용
- [ ] ApiResponse로 응답 래핑
- [ ] 적절한 HTTP 상태 코드 설정
- [ ] @Tag, @Operation 문서화 완료
- [ ] Request/Response DTO에 @Schema 추가
- [ ] 입력 검증 (@Valid, Bean Validation) 적용
- [ ] 예외 상황 고려 및 처리
- [ ] 페이징 필요 시 표준 파라미터 사용
- [ ] 한국어 메시지 사용

이러한 표준을 준수하여 일관되고 사용하기 쉬운 API를 설계하세요.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/brain1401) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
