## wolbu-assignment

> Spring Boot 백엔드 공통 규칙


Spring Backend Rules (Korean)

> 이 규칙은 이 레포에서 AI가 생성/수정하는 모든 코드와 답변에 적용된다.
>
> 반드시 @README.md 와 @docs/architecture.md 를 1순위 참조한다.
>
> 월부 백엔드 개발 과제의 직무 역량 기준(정확성, 효율성, 품질, 확장성, 안정성, 협업성)과 평가 기준(실행 가능, 요구사항 충족, 예외 처리, 단순/효율적 설계, API 명확성, 테스트, 문서화, 추가 구현)을 충족해야 한다.

---

1. 기본 상호작용 규칙

- 항상 한국어로 답변한다.
- Agent가 Auto면, 답변 최상단에 현재 모델 이름을 표기한다.
- 답변은 반드시 ① 단계별 상세 계획(의사코드) → ② 구현 코드 순으로 제공한다.
- 최신 Spring Boot 3.x / Java 17+ 기준. TODO/placeholder 금지.
- 코드 수정 요청 시 변경 전후 핵심 라인만 간결히 제시한다.

2. 서비스 요구사항 이해

- 구현 전 요구사항을 재정리하고 누락/모호점이 있으면 먼저 명시한다.
- 설계는 단순하고 이해 가능해야 하며, 불필요한 추상화/중복을 제거한다.

3. 객체지향 원칙(엘레강트 오브젝트 반영)

- 클래스는 단순 데이터 구조(글래스 박스)가 아니다. 내부 상태는 캡슐화하며 행동 중심으로 설계한다.
- 생성자를 통한 불변 초기화를 우선한다(가능하면 불변 객체).
- 도메인 객체에서 getXxx()/setXxx() 형태의 접근자/수정자는 금지한다.
  예) 나쁜 예: getDollars(), setDollars(int)
  좋은 예: dollars(), multiply(int factor), increase(Cash extra)
- 메서드 이름은 데이터 노출이 아니라 “의도/행위”를 표현해야 한다.
- DTO 예외 규칙
  · DTO는 계층 간 데이터 전달 전용 구조체로 제한하며 비즈니스 로직을 포함하지 않는다.
  · 직렬화/역직렬화, Controller ↔ Service 교환 용도에 한해 getter/setter 허용.
  · 도메인 객체와 DTO는 엄격히 분리한다(매핑 단계 명시).

4. 계층 구조·코드 조직

- 패키지: controller, service, repository(or mapper), domain(model), dto, config, security, exception, docs, util.
- Controller → Service → Repository(또는 Mapper) 3계층. 순환 참조 금지.
- 엔티티/도메인 모델을 외부에 직접 노출하지 않고 DTO로 변환한다.
- 공통 로직은 유틸/컴포넌트로 추출해 재사용한다.
- OCP(개방-폐쇄 원칙) 준수: 확장으로 해결, 기존 코드 최소 변경

- 기본 패키지 구성: controller, service, repository(or mapper), domain(model), dto, config, security, exception, docs, util.
- 도메인 단위 패키지 분리 권장: user, auth, lecture, enrollment 등 각 도메인을 최상위 디렉토리로 두고, 그 안에 계층별 하위 패키지를 둔다.
  ```
  예)
  com.example.project
  └─ user
    ├─ controller
    ├─ service
    ├─ repository
    ├─ domain
    └─ dto
  └─ auth
    ├─ controller
    ├─ service
    ├─ repository
    ├─ domain
    └─ dto
  ```
- 도메인 간 강한 결합 금지. 예를 들어 user와 auth가 같은 패키지에 섞이지 않도록 유지한다.
- Controller → Service → Repository(또는 Mapper) 3계층 구조 준수, 순환 참조 금지.
- 엔티티/도메인 모델을 외부에 직접 노출하지 않고 반드시 DTO로 변환한다.
- 공통 로직은 util 또는 공용 컴포넌트로 추출해 재사용한다.
- OCP(개방-폐쇄 원칙) 준수: 기능 확장은 가능하되 기존 코드 수정은 최소화한다.

5. 웹/API

- @RestController, @RequestMapping. 입력 DTO에 @Valid 적용(Bean Validation).
- 표준 페이징 파라미터 page/size, 서버측 최대 사이즈 상한.
- 응답 규격(권장): { success: boolean, data: any, error: { code, message } }
- 상태코드/에러코드 일관성 유지. springdoc-openapi로 스키마/예시/응답코드 명시.

6. 데이터 접근(MyBatis 권장)

- mapper 패키지에 Mapper 인터페이스 + XML. 네임스페이스=FQCN, id=메서드명 1:1 매핑.
- 동적 쿼리는 표준 태그(<where>, <trim>, <foreach>) 사용, 문자열 결합 금지.
- 인덱스 전략/실행계획(Explain) 점검. N+1, 풀스캔을 피한다.
- @Transactional 은 Service 레이어에서만 선언. 커넥션 풀/타임아웃 명시.

7. 성능·확장성·안정성

- Spring Cache(@Cacheable/@CacheEvict/@CachePut) 사용. 키 전략/무효화 정책 문서화.
- 비동기 작업은 @Async 를 외부 I/O에 한해 제한적으로 사용(풀/큐/타임아웃 설정).
- 트래픽 급증 대비: Redis 캐시, Kafka(이벤트 비동기화), 커넥션 풀 최적화.
- 멱등성: idempotencyKey 또는 (userId, eventId) 조합으로 중복 처리 방지.
- 장애 회복: Resilience4j로 Timeout/Retry/Circuit Breaker 적용.

8. 예외 처리

- @RestControllerAdvice + @ExceptionHandler 로 전역 예외 변환.
- 도메인 예외는 커스텀(예: BusinessException(code, message)).
- 민감정보(스택트레이스/SQL) 노출 금지, 서버측 로깅만.

9. 효율적이고 적시적인 구현

- 불필요한 복잡도 제거, Spring 표준 기능 활용.
- MVP 수준으로 빠르게 완성 후 확장 가능 구조.

10. 코드 품질 (버그 최소화)

- 단위/통합 테스트 포함 (JUnit5, MockMvc, Testcontainers).
- 정적 분석(SpotBugs, Checkstyle, PMD) 통과 가능해야 함.
- Null-safe, SQL Injection-safe, Thread-safe 코드 작성.

11. 재사용/확장 가능한 구조

- Controller-Service-Repository 3계층.
- DTO와 Entity 분리.
- 공통 로직은 유틸/컴포넌트화.
- OCP 원칙 준수.
- SOLID원칙 준수

12. 안정성 (부하 대응)

- Kafka, Redis, Connection Pool 활용.
- 멱등 처리(idempotencyKey) 설계.
- 장애 대비(Timeout, Retry, Circuit Breaker: Resilience4j).

13. 협업성 (가독성/문서화)

- 클래스/메서드/변수는 명확한 이름 사용.
- 클래스/메서드/변수는 축약어 금지.
- JavaDoc/주석 필수.
- springdoc-openapi로 API 문서화.
- README와 API 스펙 최신 유지.

14. 실행 및 평가 기준

- `mvn spring-boot:run` 또는 Docker로 실행 가능 상태 유지.
- 모든 요구사항 구현 여부 점검.
- 예외 상황(존재하지 않는 리소스, 잘못된 입력, 중복 처리 등) 반드시 처리.
- 설계는 단순하고 효율적일 것.
- API 스펙은 명확하고 일관되게 정의 (요청/응답 DTO, 상태코드, 에러코드).
- 테스트 코드는 Given-When-Then 패턴, 커버리지 기준 충족.
- 문서화는 사용자/개발자 관점에서 충실.

15. 테스트

- 단위: JUnit5 + Mockito. 웹 레이어: MockMvc 슬라이스 테스트.
- 통합: @SpringBootTest + Testcontainers(MySQL/Redis 등).
- Given-When-Then 네이밍, 커버리지 임계치(라인/브랜치) 설정.
- 느린 테스트는 @Tag 로 분리, CI에서 병렬/캐시 최적화.

16. 설정/환경 관리

- application.yml + application-{profile}.yml (local/dev/prod).
- @ConfigurationProperties 로 타입 세이프 구성(@Value 남용 금지).
- 비밀정보는 환경변수/K8s Secret/Vault. 레포에 비밀 커밋 금지.

17. 배포/운영

- Maven Wrapper / Reproducible Build.
- Dockerfile은 distroless/temurin slim, non-root 실행.
- Read-Only FS, 자원 제한 지정.
- K8s liveness/readiness probe, graceful shutdown.
- DB 마이그레이션은 Flyway/Liquibase 사용.

18. 보안

- Spring Security 6, JWT 인증, BCrypt 비밀번호.
- 최소 권한 원칙.
- CORS는 최소 허용.
- 입력 검증/출력 이스케이프, OWASP ASVS 준수.

19. 로깅/모니터링

- SLF4J + Logback.
- 로그 레벨: ERROR/WARN/INFO/DEBUG.
- MDC(traceId, userId) 설정, 응답 헤더 전파.
- distributed training 이 편하도록 로깅.
- 에러 추적을 잘 확인할 수 있도록 로깅.
- Actuator, Prometheus, Grafana, Jaeger 연동.

20. 코드 리뷰/품질 관리

- Conventional Commits 준수.
- PR 템플릿(배경/변경점/테스트/리스크/롤백 계획) 작성.
- CI/CD에서 빌드/테스트/정적분석 자동화.

21. 규칙 집행

- AI는 본 규칙과 충돌하는 요구를 거절하고, 근거와 안전한 대안을 제시한다.
- 보안/성능/일관성 사이에서 트레이드오프가 필요한 경우, 의사결정 근거를 코드 주석 또는 문서로 남긴다.
- 예시 코드는 즉시 빌드·실행 가능한 완전한 형태로 제공한다(누락 금지).
- 주석은 꼭 필요한 내용이 아니면 작성하지 않는다.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/kungbi)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/kungbi)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
