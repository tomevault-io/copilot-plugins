## admin-boot

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Admin-Boot is a Spring Boot 3.2.x + MyBatis-Plus backend admin system with JWT authentication and Redis caching. Provides complete RBAC (Role-Based Access Control) permission management.

## Build Commands

```bash
# Build
mvn clean package -DskipTests

# Run
mvn spring-boot:run

# Run single test
mvn test -Dtest=ClassName#methodName

# Run code generator
cd admin-generator && mvn spring-boot:run
```

## Architecture

### Multi-Module Structure
```
admin-boot
├── admin-common              # Shared utilities, configs, security
│   └── src/main/java/com/admin/common/
│       ├── annotation/       # @Log, @RequiresPermissions
│       ├── aspect/           # LogAspect (operation logging)
│       ├── config/           # Redis, Security, Web configs
│       ├── constant/         # Constants
│       ├── context/          # UserContext (current user holder)
│       ├── enums/            # ResponseCode, BusinessType, MenuType, Logical
│       ├── exception/        # BusinessException, GlobalExceptionHandler
│       ├── result/           # R<T>, PageResult
│       ├── security/          # JwtTokenUtil, LoginUser, Filters
│       └── utils/            # IpUtils, StringUtils
├── admin-system              # Business logic
│   └── src/main/
│       ├── java/com/admin/
│       │   └── system/
│       │       ├── controller/  # REST APIs
│       │       ├── service/     # IService + impl/
│       │       ├── mapper/      # MyBatis-Plus BaseMapper
│       │       ├── entity/     # Domain entities
│       │       ├── dto/        # Request DTOs
│       │       ├── vo/         # Response VOs
│       │       └── query/      # Query objects (extends PageQuery)
│       └── resources/
│           ├── application.yml   # Main config (profiles: dev/prod)
│           └── sql/             # Flyway migration scripts (V{version}__{desc}.sql)
└── admin-generator           # Code generator (MyBatis-Plus + Freemarker)
```

### Core Tech Stack
- Spring Boot 3.2.5 with MyBatis-Plus 3.5.7
- JWT (jjwt 0.12.5) - HS256, access token 15min, refresh token 7 days (one-time use)
- Redis for caching (config, dict) and token storage
- Knife4j 4.5.0 for API docs
- BCrypt password encoding

### Security Architecture
```
Request → JwtAuthenticationFilter → UserContextFilter → Controller
                                    ↓
                        @RequiresPermissions → Role/Menu validation
```

### Security Configurations
- **Redis**: Use `BasicPolymorphicTypeValidator` (not LaissezFaireSubTypeValidator) for Jackson serialization
- **JWT Secret**: Configure via `JWT_SECRET` environment variable
- **Refresh Token**: One-time use, stored by token key (not userId)
- **CORS**: Production restricts allowed-headers to specific values
- **White List**: `/api/auth/**`, `/doc.html`, `/swagger-ui/**`, `/v3/api-docs/**`, `/webjars/**`, `/actuator/health`

## API Conventions

### Unified Response
```json
{"code": 200, "message": "success", "data": {...}, "success": true}
```

### Response Codes
| Code | Meaning |
|------|---------|
| 200 | Success |
| 400 | Bad request (parameter/business error) |
| 401 | Unauthorized (invalid/expired token) |
| 403 | Forbidden (no permission) |
| 500 | System error |

### Pagination
```json
{"code": 200, "data": {"records": [], "total": 100, "size": 10, "current": 1, "pages": 10}}
```

### Query Parameters
Use `@ParameterObject` on Query DTOs to expand fields as URL parameters:
```java
@GetMapping("/list")
public R list(@ParameterObject SysUserQuery query) { }
```

## Code Generation

Configure `admin-generator/src/main/resources/generator.yml`:
```yaml
generate:
  tableNames: sys_product  # Tables to generate
controller:
  parameterObject: true     # Add @ParameterObject annotation
```

## Database

### Default Credentials
| Username | Password | Role |
|----------|----------|------|
| admin | admin123 | Super Admin |

### Key Tables (ER Relationships)
```
sys_user ─── sys_user_role ◄── sys_role ─── sys_role_menu ◄── sys_menu
                                                         ↓
                                                    sys_dict ─── sys_dict_data
```
- Core: `sys_user`, `sys_role`, `sys_menu` with junction tables
- Supporting: `sys_dict`/`sys_dict_data`, `sys_config`, `sys_logininfor`, `sys_oper_log`, `sys_file`

### Soft Delete
Column `del_flag = 1` means deleted (MyBatis-Plus logic delete configured)

## Key Annotations

| Annotation | Location | Purpose |
|-----------|----------|---------|
| `@Log` | Controller methods | Operation logging (saveParam/saveResult configurable) |
| `@RequiresPermissions` | Controller methods | Permission check |
| `@Cacheable/@CacheEvict` | Service methods | Redis caching |
| `@ParameterObject` | Query DTO parameter | Expand query fields to URL params |

## Development Notes

- **Profile**: Default is `dev`, configure via `spring.profiles.active`
- **Env Variables**: `DB_HOST`, `DB_PORT`, `DB_USERNAME`, `DB_PASSWORD`, `REDIS_HOST`, `REDIS_PORT`, `REDIS_PASSWORD`, `JWT_SECRET`
- **Build**: Uses `-parameters` compiler arg for parameter name retention (AOP logging)
- **Database Migrations**: Flyway auto-migrates on startup; scripts in `resources/sql/` with naming `V{version}__{description}.sql`
- **API Docs**: Access via `/doc.html` (Knife4j) or `/swagger-ui/index.html`

---
> Source: [wangyufei0807/admin-boot](https://github.com/wangyufei0807/admin-boot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-08 -->
