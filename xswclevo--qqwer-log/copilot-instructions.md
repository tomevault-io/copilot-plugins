## qqwer-log

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 项目概述

企业级日志分析系统，采用单体架构（已从微服务架构简化），支持大规模日志数据的实时收集、存储、查询和统计分析。

**技术栈**:
- 后端: Java 21 + Spring Boot 3.2 + MyBatis Plus + Spring Security (JWT)
- 数据库: PostgreSQL 15 (元数据) + ClickHouse 23 (日志数据，使用 MWLOGDB_ANALYSIS 库)
- 前端: Vue 3 + TypeScript + Vite + Element Plus + ECharts

## 架构设计

### 双数据库架构

使用 MyBatis Plus 动态数据源管理两个数据库：

1. **PostgreSQL (postgres)** - 元数据库

2. **ClickHouse (clickhouse)** - 日志数据库

### 数据源切换

使用 `@DS` 注解指定数据源：

```java
// Mapper 级别 - PostgreSQL
@Mapper
@DS("postgres")
public interface UserMapper extends BaseMapper<User> {}

// 方法级别 - ClickHouse
@DS("clickhouse")
public Map<String, Object> queryLogs(LogQueryRequest request) {
    // 查询 syslog 表
}
```

### 模块结构

```
log-analysis-backend/
└── log-analysis-app/              # 单体应用
    ├── auth/                       # 认证授权模块 (JWT + Spring Security)
    │   ├── config/SecurityConfig  # Security配置，白名单在此配置
    │   ├── security/              # JWT过滤器、TokenProvider
    │   └── mapper/UserMapper      # @DS("postgres")
    ├── extraction/                 # 日志提取规则模块
    │   └── mapper/                # @DS("postgres")
    ├── alert/                      # 告警规则和事件模块
    │   └── mapper/                # @DS("postgres")
    ├── config/                     # 系统配置模块
    │   └── mapper/                # @DS("postgres")
    ├── stats/                      # 统计查询模块 (核心)
    │   └── service/StatsService   # @DS("clickhouse") 查询 syslog 表
    └── common/                     # 公共模块
        ├── response/Result         # 统一响应封装
        └── exception/              # 全局异常处理
```


## 关键开发规范

### Lombok 使用

- `@Data`: 所有实体对象
- `@RequiredArgsConstructor`: Service 构造器注入
- `@Slf4j`: 日志记录
- **禁止**: 字段注入 `@Autowired`，必须使用构造器注入

### 对象转换规范

**必须使用 MapStruct 进行对象转换**，禁止手动 set/get 赋值。

#### 使用场景

1. **Request DTO → Entity**: 接收前端请求，转换为实体对象
2. **Entity → Response DTO**: 查询数据库后，转换为响应对象
3. **Entity → Entity**: 不同层次的实体对象转换
4. **复杂对象映射**: 包含嵌套对象、集合等复杂结构

#### 正确示例

```java
/**
 * Datasource 对象转换器
 */
@Mapper(componentModel = "spring")
public interface DatasourceConverter {

    /**
     * Request → Entity
     */
    Datasource toEntity(CreateDatasourceRequest request);

    /**
     * Entity → Response DTO
     */
    DatasourceDTO toDTO(Datasource entity);

    /**
     * Entity List → DTO List
     */
    List<DatasourceDTO> toDTOList(List<Datasource> entities);

    /**
     * 更新 Entity（忽略 null 字段）
     */
    @BeanMapping(nullValuePropertyMappingStrategy = NullValuePropertyMappingStrategy.IGNORE)
    void updateEntity(UpdateDatasourceRequest request, @MappingTarget Datasource entity);
}
```

#### Service 层使用

```java
@Service
@RequiredArgsConstructor
public class DatasourceService {
    private final DatasourceMapper datasourceMapper;
    private final DatasourceConverter datasourceConverter;

    public Datasource createDatasource(CreateDatasourceRequest request) {
        // ✅ 使用 MapStruct 转换
        Datasource datasource = datasourceConverter.toEntity(request);
        datasourceMapper.insert(datasource);
        return datasource;
    }

    public DatasourceDTO getDatasource(String id) {
        Datasource entity = datasourceMapper.selectById(id);
        // ✅ 使用 MapStruct 转换
        return datasourceConverter.toDTO(entity);
    }
}
```

#### 错误示例（禁止）

```java
// ❌ 禁止手动赋值
public Datasource createDatasource(CreateDatasourceRequest request) {
    Datasource datasource = new Datasource();
    datasource.setName(request.getName());
    datasource.setType(request.getType());
    datasource.setHost(request.getHost());
    // ... 大量重复代码
    return datasource;
}
```

#### MapStruct 依赖配置

确保 `pom.xml` 中已配置 MapStruct：

```xml
<dependency>
    <groupId>org.mapstruct</groupId>
    <artifactId>mapstruct</artifactId>
    <version>1.5.5.Final</version>
</dependency>
<dependency>
    <groupId>org.mapstruct</groupId>
    <artifactId>mapstruct-processor</artifactId>
    <version>1.5.5.Final</version>
    <scope>provided</scope>
</dependency>
```

#### 规范说明

1. **类型安全**: 编译时检查，避免运行时错误
2. **代码简洁**: 自动生成转换代码，无需手动维护
3. **性能优越**: 生成的代码与手写代码性能相当
4. **易于维护**: 字段变更时只需修改 Converter 接口

### HTTP 接口规范

**所有后端接口统一使用 POST 方法**，不再使用 GET/PUT/DELETE/PATCH。

#### 规范说明

1. **统一性**: 前后端交互统一使用 POST，降低理解成本
2. **安全性**: POST 请求参数在 Body 中，避免 URL 暴露敏感信息
3. **灵活性**: POST 支持复杂请求体，不受 URL 长度限制
4. **简化**: 不需要区分 RESTful 语义，降低开发复杂度

#### 正确示例

```java
@RestController
@RequestMapping("/api/datasource")
@RequiredArgsConstructor
public class DatasourceController {

    private final DatasourceService datasourceService;

    /**
     * ✅ 创建数据源 - 使用 POST
     */
    @PostMapping("/create")
    public Result<Datasource> createDatasource(@Valid @RequestBody CreateDatasourceRequest request) {
        return Result.success(datasourceService.createDatasource(request));
    }

    /**
     * ✅ 查询数据源详情 - 使用 POST（而不是 GET）
     */
    @PostMapping("/get")
    public Result<DatasourceDTO> getDatasource(@RequestBody GetDatasourceRequest request) {
        return Result.success(datasourceService.getDatasource(request.getId()));
    }

    /**
     * ✅ 更新数据源 - 使用 POST（而不是 PUT）
     */
    @PostMapping("/update")
    public Result<Datasource> updateDatasource(@Valid @RequestBody UpdateDatasourceRequest request) {
        return Result.success(datasourceService.updateDatasource(request.getId(), request));
    }

    /**
     * ✅ 删除数据源 - 使用 POST（而不是 DELETE）
     */
    @PostMapping("/delete")
    public Result<Void> deleteDatasource(@RequestBody DeleteDatasourceRequest request) {
        datasourceService.deleteDatasource(request.getId());
        return Result.success();
    }

    /**
     * ✅ 分页查询 - 使用 POST（而不是 GET）
     */
    @PostMapping("/list")
    public Result<Page<DatasourceDTO>> listDatasources(@RequestBody ListDatasourcesRequest request) {
        return Result.success(datasourceService.listDatasources(request));
    }
}
```

#### 请求 DTO 示例

```java
/**
 * 查询数据源详情请求
 */
@Data
public class GetDatasourceRequest {
    @NotBlank(message = "数据源ID不能为空")
    private String id;
}

/**
 * 删除数据源请求
 */
@Data
public class DeleteDatasourceRequest {
    @NotBlank(message = "数据源ID不能为空")
    private String id;
}

/**
 * 分页查询请求
 */
@Data
public class ListDatasourcesRequest {
    private Integer pageNum = 1;
    private Integer pageSize = 10;
    private String keyword;
    private String type;
    private String status;
}
```

#### 错误示例（禁止）

```java
// ❌ 禁止使用 GET
@GetMapping("/{id}")
public Result<Datasource> getDatasource(@PathVariable String id) {
    return Result.success(datasourceService.getDatasource(id));
}

// ❌ 禁止使用 PUT
@PutMapping("/{id}")
public Result<Datasource> updateDatasource(@PathVariable String id, @RequestBody UpdateDatasourceRequest request) {
    return Result.success(datasourceService.updateDatasource(id, request));
}

// ❌ 禁止使用 DELETE
@DeleteMapping("/{id}")
public Result<Void> deleteDatasource(@PathVariable String id) {
    datasourceService.deleteDatasource(id);
    return Result.success();
}
```

### MyBatis Plus 条件构建规范

**所有条件构建查询必须在 Mapper 中使用 default 方法实现**，不允许在 Service 层构建 Wrapper, 能写 Wrapper 构建查询语句就不要写 sql。

#### 正确示例

```java
@Mapper
@DS("postgres")
public interface AlertRuleMapper extends BaseMapper<AlertRule> {
    
    /**
     * 根据条件查询告警规则列表
     * 在 Mapper 中使用 default 方法构建 Wrapper
     */
    default List<AlertRule> selectByCondition(String name, String severity, Boolean enabled) {
        LambdaQueryWrapper<AlertRule> wrapper = new LambdaQueryWrapper<>();
        wrapper.like(StringUtils.hasText(name), AlertRule::getName, name)
               .eq(StringUtils.hasText(severity), AlertRule::getSeverity, severity)
               .eq(enabled != null, AlertRule::getEnabled, enabled)
               .orderByDesc(AlertRule::getCreatedAt);
        return selectList(wrapper);
    }
    
    /**
     * 分页查询
     */
    default Page<AlertRule> selectPageByCondition(Page<AlertRule> page, String keyword) {
        LambdaQueryWrapper<AlertRule> wrapper = new LambdaQueryWrapper<>();
        wrapper.and(StringUtils.hasText(keyword), w -> w
                .like(AlertRule::getName, keyword)
                .or()
                .like(AlertRule::getDescription, keyword))
               .orderByDesc(AlertRule::getUpdatedAt);
        return selectPage(page, wrapper);
    }
}

```

#### 错误示例（禁止）

```java
// ❌ 不要在 Service 中构建 Wrapper
@Service
public class AlertRuleService {
    public List<AlertRule> queryRules(String name) {
        LambdaQueryWrapper<AlertRule> wrapper = new LambdaQueryWrapper<>();
        wrapper.like(AlertRule::getName, name);  // 错误：在 Service 中构建
        return alertRuleMapper.selectList(wrapper);
    }
}
```

#### 规范说明

1. **封装性**: Mapper 负责数据访问逻辑，包括条件构建
2. **可测试性**: default 方法可以直接测试，无需 Mock Service
3. **复用性**: 多个 Service 可以复用同一个 Mapper 方法
4. **清晰性**: Service 层只关注业务逻辑，不涉及查询细节
5. **类型安全**: 使用 LambdaQueryWrapper 避免字符串硬编码

#### Service 层调用

```java
@Service
@RequiredArgsConstructor
public class AlertRuleService extends ServiceImpl<AlertRuleMapper, AlertRuleEntity> {

    public List<AlertRule> getRules(String name, String severity, Boolean enabled) {
        // 直接调用 Mapper 的 default 方法
        return getBaseMapper().selectByCondition(name, severity, enabled);
    }

    public Page<AlertRule> getRulesPage(int pageNum, int pageSize, String keyword) {
        Page<AlertRule> page = new Page<>(pageNum, pageSize);
        return getBaseMapper().selectPageByCondition(page, keyword);
    }
}
```

### Repository 层引入标准

**默认策略**：使用 Mapper 的 default 方法处理数据访问逻辑，符合 KISS 和 YAGNI 原则。

**何时引入 Repository 层**（满足以下任一条件时考虑引入）：

1. **代码行数**：Mapper 的 default 方法超过 30 行
   - 查询逻辑过于复杂，影响可读性
   - 需要多次条件判断或循环处理

2. **依赖数量**：需要注入其他 Mapper 或 Service
   - 需要组合多个 Mapper 的查询结果
   - 需要调用其他 Service 的业务逻辑

3. **业务逻辑**：包含 if-else、循环等业务判断
   - 数据访问逻辑中混入了业务规则
   - 需要根据业务条件动态选择查询策略

4. **跨数据源**：需要同时查询 PostgreSQL 和 ClickHouse
   - 需要组合元数据（PostgreSQL）和日志数据（ClickHouse）
   - 需要在两个数据源之间做数据关联

5. **重复出现**：3 次以上类似的复杂查询
   - 多个 Service 需要相同的复杂查询逻辑
   - 需要统一封装以提高复用性

#### Repository 层示例

```java
@Repository
@RequiredArgsConstructor
public class AlertRepository {
    private final AlertRuleMapper ruleMapper;
    private final AlertEventMapper eventMapper;
    private final StatsService statsService;  // ClickHouse 查询

    /**
     * 复杂查询：获取规则及其触发统计（跨 PostgreSQL 和 ClickHouse）
     */
    public AlertRuleWithStats getRuleWithStats(Long ruleId, String timeRange) {
        // 从 PostgreSQL 查询规则
        AlertRule rule = ruleMapper.selectById(ruleId);

        // 从 PostgreSQL 查询事件
        List<AlertEvent> events = eventMapper.selectByRuleId(ruleId);

        // 从 ClickHouse 查询日志统计
        Map<String, Object> logStats = statsService.queryLogsByRule(ruleId, timeRange);

        // 组合结果
        return AlertRuleWithStats.builder()
            .rule(rule)
            .events(events)
            .logStats(logStats)
            .build();
    }
}
```

#### 架构演进路径

```
当前架构：Controller → Service → Mapper → Database

引入 Repository 后：Controller → Service → Repository → Mapper → Database
                                              ↓
                                         (组合多个 Mapper)
```

**重要原则**：
- ✅ 优先使用 Mapper default 方法（简单查询）
- ✅ 按需引入 Repository 层（复杂场景）
- ✅ 避免过度设计，保持架构简洁
- ❌ 不要为了分层而分层

### API 开发

所有 Controller 返回统一 `Result<T>`:

```java
@PostMapping("/logs/query")
public Result<Map<String, Object>> queryLogs(@Valid @RequestBody LogQueryRequest request) {
    return Result.success(statsService.queryLogs(request));
}
```

### Security 配置

白名单配置在 `SecurityConfig.java:53-61`:

```java
.requestMatchers(
    "/api/auth/login",
    "/api/auth/refresh",
    "/api/stats/**",  // 统计接口已开放
    "/actuator/health",
    "/actuator/info"
).permitAll()
```

添加新的公开端点需修改此配置并重启应用。

### ClickHouse 查询注意事项

1. **表名**: 使用 `syslog`（不是 log_entries）
2. **数据库**: `MWLOGDB_ANALYSIS`
3. **字段映射**: 使用 `StatsService.mapDimensionField()` 方法
4. **时间范围**: 始终使用 `timestamp >= ? AND timestamp <= ?`
5. **性能**: 利用已有索引（severity, hostname, appname, source_type）
6. **可用字段**: 仅支持以下字段
   - `id`, `severity`, `hostname`, `appname`, `source_type`
   - `message`, `facility`, `procid`, `source_ip`, `timestamp`, `raw`
   - **不支持**: `status`, `code`, `user` 等自定义字段

### 常见问题处理

**403 错误**: 检查 SecurityConfig 白名单，重启应用生效
**Lombok 编译错误**: 确保 Lombok 版本 1.18.34，检查 IDE 插件
**ClickHouse 连接失败**: 检查数据库名是否为 `MWLOGDB_ANALYSIS`
**数据源切换失败**: 确认 `@DS` 注解正确，检查动态数据源配置

## 环境配置

### 默认配置

- **后端端口**: 8080
- **PostgreSQL**: localhost:5432/postgres (postgres/123456)
- **ClickHouse**: 10.180.5.72:8123/MWLOGDB_ANALYSIS (default/mwclickhouse@2024)
- **Redis**: localhost:6379
- **Kafka**: 10.18.5.186:9092

- **postgres**: 所采用 docker部署，你可以操作，clickhouse 为远程服务器，你不可以操作。

### 环境变量覆盖

```bash
# PostgreSQL
export DB_HOST=localhost
export DB_PORT=5432
export DB_NAME=postgres
export DB_USER=postgres
export DB_PASSWORD=123456

# ClickHouse
export CLICKHOUSE_HOST=10.180.5.72
export CLICKHOUSE_PORT=8123
export CLICKHOUSE_DB=MWLOGDB_ANALYSIS
export CLICKHOUSE_USER=default
export CLICKHOUSE_PASSWORD=mwclickhouse@2024
```

## 重要文档参考

- **动态数据源**: `log-analysis-backend/DYNAMIC_DATASOURCE_GUIDE.md`
- **ClickHouse 迁移**: `log-analysis-backend/CLICKHOUSE_MIGRATION_GUIDE.md`
- **后端实现**: `log-analysis-backend/BACKEND_IMPLEMENTATION.md`
- **403 错误修复**: `log-analysis-backend/FIX_403_ERROR.md`
- **接口文档**: `后端每次更新的接口文档都需要在当前new-log目录下生成一份名为SWAGGER.md 文档, 前端每次对接新接口都要从此文档里面读取最新的接口结构,从而调整代码;如果没有找到该文件则去查看对应的*controller文件` 
## Git 提交规范

遵循 Conventional Commits:

- `feat:` 新功能
- `fix:` Bug 修复
- `refactor:` 重构
- `docs:` 文档更新
- `test:` 测试相关
- `chore:` 构建/工具链更新

示例: `feat: add dynamic datasource support for PostgreSQL and ClickHouse`

## 接口文档

- **前端文档目录**: `docs/frontend/xxx.md`
- **后端文档目录**: `docs/backend-api/xxx.md` 中
- **后端 API 参考文档**: `docs/backend/API_REFERENCE.md` - 所有接口的统一索引
- **前端读取文档目录**: `docs/backend-api/xxx.md` 读取该目录下的接口文档，并且根据接口文档调用接口
- **注意**: 前后端每次修改都需要在 docs/各自的文档中真难吃本次的修改，方便后续对接修改

### 接口文档同步规范

**重要**: 所有新增的后端接口都必须同步更新到 `docs/backend/API_REFERENCE.md` 文档中。

更新内容包括：
1. 接口路径（完整 URL）
2. HTTP 方法（GET/POST/PUT/DELETE/PATCH）
3. 接口作用说明
4. 代码位置（Controller 文件路径和行号）
5. 所属模块分类

示例格式：
```markdown
| 接口路径 | HTTP方法 | 作用 | 代码位置 |
|---------|---------|------|---------|
| `/api/example/test` | POST | 测试接口 | ExampleController.java:50 |
```

## 注意

- 不允许 随意写文档与脚本, 除非得到使用者的明确命令

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/XSWClevo) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
