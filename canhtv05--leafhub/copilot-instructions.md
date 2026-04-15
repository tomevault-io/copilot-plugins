## leafhub

> This is a microservices-based Spring Boot application with the following modules:

# Cursor Rules for MS Project

## Project Overview
This is a microservices-based Spring Boot application with the following modules:
- **auth**: Authentication and authorization service (JPA/PostgreSQL)
- **profile**: User profile service (Neo4j)
- **file**: File upload and management service (MongoDB)
- **notification**: Email notification service
- **graphql-bff**: GraphQL BFF layer using Netflix DGS
- **common**: Shared common code and gRPC definitions
- **gateway**: API Gateway

## Code Style & Conventions

### Naming Conventions

#### Domain Entity Naming
- Use PascalCase: `User`, `UserProfile`, `Interest`, `File`

#### DTO Naming
- **General DTO**: `{Domain}DTO` (e.g., `UserDTO`, `InterestDTO`, `UserProfileDTO`)
- **Request DTO**: `Update{Domain}{Field}Req` or `{Action}{Domain}{Field}Req` (e.g., `UpdateBioProfileReq`, `ChangeCoverByUrlReq`, `UserProfileCreationReq`)
- **Response DTO**: `{Domain}Res` or `Update{Domain}{Field}Res` (e.g., `UserProfileRes`, `UserRes`, `FileRes`)

#### Service Naming
- **Service**: `{Domain}Service` (e.g., `UserService`, `UserProfileService`, `InterestService`, `FileService`)

#### Repository Naming
- **Repository**: `{Domain}Repository` (e.g., `UserRepository`, `UserProfileRepository`, `InterestRepository`, `FileRepository`)

### Package Structure
- Follow the pattern: `com.leaf.{module}.{layer}`
- Layers: `controller`, `service`, `repository`, `domain`, `dto`, `grpc`, `config`, `security`, `exception`
- DTOs should be in subpackages: `dto.req`, `dto.res`, `dto.search`, `dto.excel` when applicable

### Service Layer Patterns

#### Service Class Structure
```java
@Slf4j
@Service
@RequiredArgsConstructor
@Transactional(propagation = Propagation.REQUIRED, rollbackFor = Exception.class)
public class UserService {
    // Dependencies injected via constructor (Lombok @RequiredArgsConstructor)
    private final UserRepository repository;
    private final OtherService otherService;

    // Write operations
    public UserDTO createUser(UserProfileCreationReq req) {
        // Implementation
    }

    // Read operations
    @Transactional(readOnly = true)
    public UserDTO getUser(Long id) {
        // Implementation
    }
}
```

**Naming**: Service class name follows pattern `{Domain}Service` (e.g., `UserService`, `UserProfileService`, `InterestService`)

#### Transaction Management
- **Class-level**: Use `@Transactional(propagation = Propagation.REQUIRED, rollbackFor = Exception.class)` for services with write operations
  - Import: `import org.springframework.transaction.annotation.Propagation;`
  - Write operations will inherit transaction settings from class-level annotation
- **Write operations**: No need to add `@Transactional` annotation to write methods (inherit from class-level)
- **Read operations**: Always add `@Transactional(readOnly = true)` to each read method to override class-level settings
- **Propagation**: Use `REQUIRED` (default) for most cases

#### Exception Handling
- Always use `ApiException` with `ErrorMessage` enum
- Pattern: `throw new ApiException(ErrorMessage.XXX)`
- Use `SecurityUtils.getCurrentUserLogin().orElseThrow(() -> new ApiException(ErrorMessage.UNAUTHENTICATED))`
- Use `repository.findById(id).orElseThrow(() -> new ApiException(ErrorMessage.XXX_NOT_FOUND))`

### Controller Layer Patterns

#### REST Controller Structure
```java
@RestController
@RequiredArgsConstructor
public class UserController {
    private final UserService service;

    @GetMapping("/users")
    public ResponseEntity<ResponseObject<UserDTO>> getUser() {
        return ResponseEntity.ok(ResponseObject.success(service.getUser()));
    }

    @PostMapping("/users")
    public ResponseEntity<ResponseObject<UserDTO>> createUser(@RequestBody UserProfileCreationReq req) {
        return ResponseEntity.ok(ResponseObject.success(service.createUser(req)));
    }
}
```

#### Response Format
- Always return `ResponseEntity<ResponseObject<T>>`
- Use `ResponseObject.success(data)` for success responses
- Use `ResponseObject.success(data, pageResponse)` for paginated responses
- Use `ResponseObject.error(ErrorMessage.XXX)` for error responses

### DTO Patterns

#### DTO Requirements
- **All DTOs must implement `Serializable`**
- **All DTOs must have `private static final long serialVersionUID = 1L;`**
- **General DTOs (e.g., `UserDTO`, `UserProfileDTO`) must have static factory method `toDTO(Entity entity)` using `BeanUtils.copyProperties`**
- **Response DTOs must have static factory method using `BeanUtils.copyProperties`**

#### General DTO Structure
```java
@Data
@AllArgsConstructor
@NoArgsConstructor
@Builder
public class UserDTO implements Serializable {
    private static final long serialVersionUID = 1L;

    private Long id;
    private String username;
    private String email;

    // Static factory method for entity conversion using BeanUtils
    public static UserDTO toDTO(User entity) {
        UserDTO dto = new UserDTO();
        BeanUtils.copyProperties(entity, dto);
        return dto;
    }
}
```

**Note**: Don't forget to import: `import org.springframework.beans.BeanUtils;`

**Naming**: General DTO follows pattern `{Domain}DTO` (e.g., `UserDTO`, `InterestDTO`, `UserProfileDTO`)

#### Response DTO Structure
```java
@Data
@AllArgsConstructor
@NoArgsConstructor
@Builder
public class UserProfileRes implements Serializable {
    private static final long serialVersionUID = 1L;

    private String id;
    private String userId;
    private String fullname;

    // Static factory method for entity conversion using BeanUtils
    public static UserProfileRes toUserProfileRes(UserProfile entity) {
        UserProfileRes res = new UserProfileRes();
        BeanUtils.copyProperties(entity, res);
        return res;
    }
}
```

**Note**: Don't forget to import: `import org.springframework.beans.BeanUtils;`

**Naming**: Response DTO follows pattern `{Domain}Res` or `Update{Domain}{Field}Res` (e.g., `UserProfileRes`, `UserRes`, `FileRes`)

#### Request DTO Structure
```java
@Data
@AllArgsConstructor
@NoArgsConstructor
@Builder
public class UpdateBioProfileReq implements Serializable {
    private static final long serialVersionUID = 1L;

    @NotBlank
    private String bio;
}
```

**Naming**: Request DTO follows pattern `Update{Domain}{Field}Req` or `{Action}{Domain}{Field}Req` (e.g., `UpdateBioProfileReq`, `ChangeCoverByUrlReq`, `UserProfileCreationReq`)

**Note**: Request DTOs do NOT have `toEntity()` method. Only General DTOs (e.g., `UserDTO`, `UserProfileDTO`) have static factory method `toDTO(Entity entity)`.

#### Search Req/Res
- Use `SearchRequest` record from `com.leaf.common.dto.search`
- Use `SearchResponse<T>` from `com.leaf.common.dto.search`
- Convert to `Pageable` using `searchRequest.toPageable()`
- Build `PageResponse` from Spring `Page` object:
```java
PageResponse pageResponse = PageResponse.builder()
    .currentPage(page.getNumber() + 1)
    .size(page.getSize())
    .total(page.getTotalElements())
    .totalPages(page.getTotalPages())
    .count(page.getContent().size())
    .build();
return new SearchResponse<>(content, pageResponse);
```

### Repository Patterns

#### JPA Repository (PostgreSQL - auth module)
```java
@Repository
public interface UserRepository extends JpaRepository<User, Long>, JpaSpecificationExecutor<User> {
    Optional<User> findByUsername(String username);

    // Custom query methods
    @Query("SELECT u FROM User u WHERE u.email = :email")
    Optional<User> findByEmail(@Param("email") String email);
}
```

**Naming**: Repository interface follows pattern `{Domain}Repository` (e.g., `UserRepository`, `UserProfileRepository`, `InterestRepository`)

#### Neo4j Repository (profile module)
```java
@Repository
public interface UserProfileRepository extends Neo4jRepository<UserProfile, String> {
    Optional<UserProfile> findByUserId(String userId);

    @Query("MATCH (up:UserProfile {field: $field}) RETURN up")
    Optional<UserProfile> findCustom(@Param("field") String field);
}
```

**Naming**: Repository interface follows pattern `{Domain}Repository` (e.g., `UserRepository`, `UserProfileRepository`, `InterestRepository`)

#### MongoDB Repository (file module)
```java
@Repository
public interface FileRepository extends MongoRepository<File, String> {
    List<File> findByOwnerId(String ownerId);
}
```

**Naming**: Repository interface follows pattern `{Domain}Repository` (e.g., `UserRepository`, `UserProfileRepository`, `FileRepository`)

### Mapper Patterns

#### General Mapper Rules
- **DO NOT use MapStruct** - Use manual mapping, BeanUtils.copyProperties, or Lombok Builder instead
- **All mappers must use singleton pattern** with a single instance
- **Mapper class naming**: `{Domain}Mapper` or `{Domain}GrpcMapper` (e.g., `UserProfileMapper`, `UserProfileGrpcMapper`)
- **Method naming**: `toXxx()` for conversion methods (e.g., `toUserProfileDTO()`, `toGrpcResponse()`)

#### Mapper Implementation Options
1. **Manual mapping** - Direct field assignment using Lombok Builder or setters
2. **BeanUtils.copyProperties** - For simple field-to-field mapping
3. **Lombok Builder** - For complex mapping with transformations

#### Mapper Singleton Pattern
```java
public class XxxMapper {
    private static final XxxMapper INSTANCE = new XxxMapper();

    private XxxMapper() {}

    public static XxxMapper getInstance() {
        return INSTANCE;
    }

    // Manual mapping example
    public XxxDTO toXxxDTO(XxxEntity entity) {
        if (entity == null) {
            return null;
        }
        return XxxDTO.builder()
            .id(entity.getId())
            .name(entity.getName())
            .build();
    }

    // BeanUtils mapping example
    public XxxDTO toXxxDTO(XxxEntity entity) {
        if (entity == null) {
            return null;
        }
        XxxDTO dto = new XxxDTO();
        BeanUtils.copyProperties(entity, dto);
        return dto;
    }
}
```

**Note**: Don't forget to import: `import org.springframework.beans.BeanUtils;`
```

### gRPC Service Patterns

#### gRPC Service Implementation
```java
@GrpcService
@RequiredArgsConstructor
public class XxxGrpcServiceImpl extends XxxGrpcServiceGrpc.XxxGrpcServiceImplBase {
    private final XxxService service;
    private final XxxGrpcMapper mapper = XxxGrpcMapper.getInstance();

    @Override
    public void getXxx(XxxReq req, StreamObserver<XxxRes> responseObserver) {
        var dto = service.getXxx(req.getId());
        var res = mapper.toGrpcRes(dto);
        responseObserver.onNext(res);
        responseObserver.onCompleted();
    }
}
```

#### gRPC Mapper Pattern
- Use singleton pattern with `getInstance()` - single instance only
- Methods: `toGrpcXxx()`, `toXxx()` for bidirectional conversion
- Can use manual mapping, BeanUtils.copyProperties, or Lombok Builder
- **DO NOT use MapStruct**

### GraphQL Patterns (graphql-bff module)

#### GraphQL Resolver Structure
```java
@DgsComponent
@RequiredArgsConstructor
public class XxxQueryResolver {
    private final XxxGrpcClient grpcClient;
    private final XxxMapper mapper = XxxMapper.getInstance();

    @DgsQuery(field = "xxx")
    public Mono<XxxDTO> xxx(String id) {
        return Mono.fromCallable(() -> grpcClient.getXxx(id))
            .subscribeOn(Schedulers.boundedElastic())
            .flatMap(response -> Mono.justOrEmpty(mapper.toXxxDTO(response)));
    }

    @DgsData(parentType = "XxxDTO", field = "nested")
    public Mono<NestedDTO> nested(DgsDataFetchingEnvironment dfe) {
        XxxDTO parent = dfe.getSource();
        return Mono.fromCallable(() -> grpcClient.getNested(parent.getId()))
            .subscribeOn(Schedulers.boundedElastic())
            .map(mapper::toNestedDTO);
    }
}
```

**Note**: Mapper must use singleton pattern with `getInstance()`. Use manual mapping, BeanUtils.copyProperties, or Lombok Builder. **DO NOT use MapStruct**.

### Kafka Patterns

#### Kafka Listener
```java
@Slf4j
@Service
@RequiredArgsConstructor
public class KafkaListenerService {
    private final XxxService service;

    @KafkaListener(
        topics = EventConstants.XXX_TOPIC,
        groupId = EventConstants.XXX_GROUP_ID,
        containerFactory = "xxxKafkaListenerContainerFactory"
    )
    public void handleXxxEvent(XxxEvent event) {
        try {
            service.processEvent(event);
        } catch (Exception e) {
            log.error("Failed to process event: {}", event, e);
        }
    }
}
```

#### Kafka Producer
```java
@Service
@RequiredArgsConstructor
public class KafkaProducerService {
    private final KafkaTemplate<String, Object> kafkaTemplate;

    public void send(String topic, Object event) {
        kafkaTemplate.send(topic, event);
    }
}
```

### Security Patterns

#### Authentication Check
```java
String username = SecurityUtils.getCurrentUserLogin()
    .orElseThrow(() -> new ApiException(ErrorMessage.UNAUTHENTICATED));
```

#### Authorization Check
```java
if (SecurityUtils.isGlobalUser()) {
    // Admin-only logic
}
```

### Caching Patterns

#### Redis Cache Eviction
```java
String cacheKey = CacheKey.XXX.name() + ":" + id;
redisService.evict(cacheKey);
```

### Common Utilities

#### Validation
- Use Jakarta Validation annotations: `@NotBlank`, `@NotNull`, `@Email`, etc.
- Validate in service layer before processing

#### Date/Time Handling
- Use `Instant` for timestamps
- Use `InstantToStringSerializer` for JSON serialization
- Use `DateUtils` from `com.leaf.common.utils` for date formatting

#### Common Utils
- `CommonUtils.isEmpty()` for null/empty checks
- `CommonUtils.isNotEmpty()` for non-empty checks
- `JsonF` for JSON operations

### Error Handling

#### Exception Translator
Each module should have an `ExceptionTranslator`:
```java
@RestControllerAdvice
public class ExceptionTranslator {
    @ExceptionHandler(ApiException.class)
    public ResponseEntity<ResponseObject<Void>> handleApiException(ApiException ex) {
        return ResponseEntity
            .status(HttpStatus.BAD_REQUEST)
            .body(ResponseObject.error(ex.getErrorMessage()));
    }
}
```

### Lombok Usage

#### Standard Annotations
- `@RequiredArgsConstructor` for dependency injection
- Use `private final` for immutable fields (no @FieldDefaults)
- `@Slf4j` for logging
- `@Data` or `@Getter/@Setter` for DTOs
- `@Builder` or `@SuperBuilder` for DTOs
- `@NoArgsConstructor` and `@AllArgsConstructor` for DTOs

### File Upload Patterns

#### MultipartFile Handling
```java
public FileRes upload(MultipartFile[] files, ResourceType resourceType) {
    String userId = SecurityUtils.getCurrentUserLogin()
        .orElseThrow(() -> new ApiException(ErrorMessage.UNAUTHENTICATED));

    // Process files
    for (MultipartFile file : files) {
        // Validate content type
        // Process upload
    }

    return saveFileMetadata(userId, files);
}
```

### Database-Specific Patterns

#### JPA Specifications (for complex queries)
```java
private Specification<Xxx> createSpecification(SearchRequest criteria) {
    return (root, query, cb) -> {
        List<Predicate> predicates = new ArrayList<>();
        if (StringUtils.isNotBlank(criteria.searchText())) {
            predicates.add(cb.like(cb.lower(root.get("field")),
                "%" + criteria.searchText().toLowerCase() + "%"));
        }
        return cb.and(predicates.toArray(new Predicate[0]));
    };
}
```

#### Neo4j Cypher Queries
- Use multi-line strings with `"""` for readability
- Use `@Param` for parameters
- Return DTOs directly from queries when possible

#### MongoDB Aggregation
- Use `MongoTemplate` for complex aggregations
- Use `Aggregation` pipeline builder

### Testing Patterns

#### Unit Test Structure
```java
@ExtendWith(MockitoExtension.class)
class UserServiceTest {
    @Mock
    UserRepository repository;

    @InjectMocks
    UserService service;

    @Test
    void shouldCreateUser() {
        // Given
        // When
        // Then
    }
}
```

## Code Generation Rules

### When Creating New Service
1. Create domain entity in `domain` package (e.g., `User`, `UserProfile`, `Interest`)
2. Create repository interface extending appropriate base repository
   - **Naming**: `{Domain}Repository` (e.g., `UserRepository`, `UserProfileRepository`, `InterestRepository`)
3. Create DTOs (Req/Res) in `dto` package
   - **General DTO**: `{Domain}DTO` (e.g., `UserDTO`, `InterestDTO`, `UserProfileDTO`)
     - Must have static factory method `toDTO(Entity entity)` using `BeanUtils.copyProperties`
   - **Request DTO**: `Update{Domain}{Field}Req` or `{Action}{Domain}{Field}Req` (e.g., `UpdateBioProfileReq`, `ChangeCoverByUrlReq`, `UserProfileCreationReq`)
     - Do NOT have `toEntity()` or `toDTO()` method
   - **Response DTO**: `{Domain}Res` or `Update{Domain}{Field}Res` (e.g., `UserProfileRes`, `UserRes`, `FileRes`)
     - Must have static factory method using `BeanUtils.copyProperties`
   - All DTOs must implement `Serializable` with `serialVersionUID = 1L`
4. Create service class with `@Service`, `@RequiredArgsConstructor`, and `@Transactional(propagation = Propagation.REQUIRED, rollbackFor = Exception.class)`
   - **Naming**: `{Domain}Service` (e.g., `UserService`, `UserProfileService`, `InterestService`)
5. Write methods will inherit transaction settings from class-level annotation (no need to add `@Transactional` to write methods)
6. Add `@Transactional(readOnly = true)` to each read method
7. Create controller with `@RestController`
8. Add exception handling in `ExceptionTranslator`
9. Add security configuration if needed

### When Creating New gRPC Service
1. Define proto file in `common/src/main/proto`
2. Generate Java classes (auto-generated)
3. Create mapper class with singleton pattern
   - Use `getInstance()` method with single instance
   - Use manual mapping, BeanUtils.copyProperties, or Lombok Builder
   - **DO NOT use MapStruct**
4. Create gRPC service implementation
5. Register in application configuration

### When Creating New GraphQL Resolver
1. Define GraphQL schema in `.graphqls` file
2. Create resolver class with `@DgsComponent`
3. Use reactive `Mono` for async operations
4. Use `Schedulers.boundedElastic()` for blocking calls
5. Create mapper for gRPC to DTO conversion
   - Use singleton pattern with `getInstance()`
   - Use manual mapping, BeanUtils.copyProperties, or Lombok Builder
   - **DO NOT use MapStruct**

## Best Practices

1. **Always validate input** in service layer before processing
2. **Use Optional** for nullable returns from repositories
3. **Use static factory methods with BeanUtils.copyProperties** in Response DTOs (Res) for entity conversion
4. **Use static factory method `toDTO(Entity entity)` with BeanUtils.copyProperties** in General DTOs (e.g., `UserDTO`, `UserProfileDTO`) for entity conversion
5. **Request DTOs (Req) do NOT have `toEntity()` or `toDTO()` method** - only General DTOs have `toDTO()` method
6. **All DTOs must implement Serializable** with `serialVersionUID = 1L`
7. **Always add @Transactional annotation** to read service methods: `@Transactional(readOnly = true)` (write methods inherit from class-level annotation)
8. **Use singleton pattern for mappers** - All mappers must use `getInstance()` with single instance
9. **Use manual mapping, BeanUtils.copyProperties, or Lombok Builder** for mapper implementations - **DO NOT use MapStruct**
10. **Log errors** with context using `log.error("message: {}", variable, exception)`
11. **Use constants** from `EventConstants` for Kafka topics
12. **Use enums** from `ErrorMessage` for error handling
13. **Cache keys** should use `CacheKey` enum
14. **Security checks** should be done early in service methods
15. **Transaction boundaries** should be clear and minimal
16. **Use reactive programming** in GraphQL resolvers for better performance

## Anti-Patterns to Avoid

1. ❌ Don't use `@Transactional` on controllers
2. ❌ Don't use `@Transactional` on private methods
3. ❌ Don't create DTOs without implementing `Serializable` and `serialVersionUID`
4. ❌ Don't use MapStruct for mapping (use manual mapping, BeanUtils.copyProperties, or Lombok Builder)
5. ❌ Don't create mappers without singleton pattern (must use `getInstance()` with single instance)
6. ❌ Don't forget to add `@Transactional(readOnly = true)` to read service methods
7. ❌ Don't add `@Transactional` to DTOs (DTOs are data transfer objects, not service classes)
8. ❌ Don't use MapStruct for mapping (use manual mapping, BeanUtils.copyProperties, or Lombok Builder)
9. ❌ Don't create mappers without singleton pattern (must use `getInstance()` with single instance)
10. ❌ Don't catch and swallow exceptions without logging
11. ❌ Don't use `null` checks when `Optional` is available
12. ❌ Don't hardcode error messages (use `ErrorMessage` enum)
13. ❌ Don't mix database technologies in same service
14. ❌ Don't expose entity classes directly (always use DTOs)
15. ❌ Don't use blocking operations in GraphQL resolvers without `Schedulers`
16. ❌ Don't use `@FieldDefaults` annotation (use `private final` explicitly)

## Module-Specific Notes

### auth module
- Uses JPA/PostgreSQL
- Handles authentication, authorization, user management
- Uses `JpaSpecificationExecutor` for dynamic queries

### profile module
- Uses Neo4j for graph database
- Handles user profiles, relationships, interests
- Uses Cypher queries for complex graph operations

### file module
- Uses MongoDB for file metadata
- Integrates with Cloudinary for file storage
- Handles images and videos

### notification module
- Handles email notifications
- Uses Kafka for event-driven architecture
- Integrates with email service

### graphql-bff module
- Uses Netflix DGS framework
- Aggregates data from multiple microservices via gRPC
- Uses reactive programming (Mono/Flux)

### common module
- Contains shared DTOs, exceptions, utilities
- Contains gRPC proto definitions
- Contains framework code

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/canhtv05) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
