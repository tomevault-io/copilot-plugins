## java-backend-mybatis-plus

> 易研 OA 后端项目 - Java + MyBatis Plus 开发规范


# 易研发 OA 后端开发规范 (MyBatis Plus)

## 1. 实体类规范 (Entity)

为了确保系统的审计追踪和逻辑删除功能正常运行，所有业务实体类必须遵循以下规范：

### 1.1 继承要求
- **强制要求**：所有的业务实体类（Entity）必须继承自 `com.cac.yiyan.mybatis.domain.BaseEntity`。
- **作用**：`BaseEntity` 封装了创建人、创建时间、更新人、更新时间、更新 IP 以及逻辑删除等基础字段。

### 1.2 主键策略
- 使用 MyBatis Plus 的 `IdType.ASSIGN_ID`（雪花算法）或根据项目具体配置。

---

## 2. 数据库建表规范 (MySQL)

所有业务表必须包含以下基础审计字段，以适配后端 `BaseEntity` 和自动填充拦截器：

| 字段名 | 类型 | 说明 | 默认值 / 备注 |
| :--- | :--- | :--- | :--- |
| `creator` | `VARCHAR(64)` | 创建人 | 记录创建人工号或 ID |
| `create_time` | `DATETIME` | 创建时间 | `DEFAULT CURRENT_TIMESTAMP` |
| `updater` | `VARCHAR(64)` | 更新人 | 记录最后一次修改人 |
| `update_time` | `DATETIME` | 更新时间 | `DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP` |
| `update_ip` | `VARCHAR(255)` | 最后更新 IP | 记录操作者的 IP 地址 |
| `is_deleted` | `TINYINT(2)` | 逻辑删除标记 | `DEFAULT 0` (0: 正常, 1: 已删除) |

### 2.1 建表 SQL 示例

```sql
CREATE TABLE `your_table_name` (
  `id` BIGINT NOT NULL COMMENT '主键ID',
  -- 业务字段...
  `creator` VARCHAR(64) DEFAULT NULL COMMENT '创建者',
  `create_time` DATETIME DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
  `updater` VARCHAR(64) DEFAULT NULL COMMENT '更新者',
  `update_time` DATETIME DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新时间',
  `update_ip` VARCHAR(255) DEFAULT NULL COMMENT '更新IP',
  `is_deleted` TINYINT(2) DEFAULT 0 COMMENT '是否删除',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='业务表说明';
```

---

## 3. 业务逻辑规范

- **自动填充**：由于继承了 `BaseEntity`，在执行 `insert` 或 `update` 操作时，无需手动设置 `create_time`、`updater` 等字段。MyBatis Plus 的 `MetaObjectHandler` 会自动拦截并填充。
- **逻辑删除**：查询时 MyBatis Plus 会自动过滤 `is_deleted = 1` 的记录。执行 `deleteById` 时会执行 `UPDATE` 操作而非 `DELETE`。

---

## 4. 控制器分页查询规范 (Controller)

为了保持前后端交互的一致性及后端代码的可维护性，所有分页查询必须遵循下述标准（即 `PageParam` + `PageData` 模式）：

### 4.1 查询参数 (Query Object)
- **强制要求**：分页查询参数类必须继承自 `com.cac.yiyan.common.page.PageParam`。
- **示例**：
  ```java
  public class YourQuery extends PageParam {
      @ApiModelProperty("过滤条件")
      private String keyword;
  }
  ```

### 4.2 接口返回结构 (Response)
- **强制要求**：分页接口必须返回 `Result<PageData<T>>`。
- **字段要求**：`PageData` 必须包含 `list` (数据集合) 和 `total` (总记录数)。
- **示例**：
  ```java
  @GetMapping("/page")
  @ApiOperation("分页查询列表")
  public Result<PageData<YourVO>> page(YourQuery query) {
      return Result.ok(yourService.getPage(query));
  }
  ```

### 4.3 Mapper 层规范
- **强制要求**：Mapper 接口必须继承自 `com.cac.yiyan.mybatis.mapper.BaseMapperX<T>`。
- **作用**：`BaseMapperX` 封装了返回 `PageData` 的 `selectPage` 方法，确保返回字段符合前端要求的 `list` / `total` 规范，避免使用 MyBatis Plus 原生的 `IPage` (records) 结构。

---

## 5. 代码编写规范

### 5.1 类的引入规范 (Imports)
- **强制要求**：在代码逻辑中，**严禁使用类的全限定名**（例如 `java.util.Map<String, Object> map = new java.util.HashMap<>();`）。
- **要求**：所有的类都必须在文件顶部通过 `import` 语句统一引入。
- **目的**：保持代码的整洁与高可读性，避免代码过长拥挤。
- **代码示例**：
  ```java
  // ❌ 错误示范：在方法或变量中直接写死完整包名
  public java.util.Map<String, Object> getStats() {
      java.util.Map<String, Object> map = new java.util.HashMap<>();
  }

  // ✅ 正确示范：顶部 import 导入依赖
  import java.util.Map;
  import java.util.HashMap;

  public Map<String, Object> getStats() {
      Map<String, Object> map = new HashMap<>();
  }
  ```

---

## 6. 业务与架构设计规范 (Best Practices)

### 6.1 禁用直接/复杂 SQL 编程
- **强制要求**：**严禁在 Mapper / XML 中手写复杂的联表 (JOIN) SQL 语句！**
- **开发思维推荐**：多依托和使用 MyBatis-Plus 提供的包装器、基础方法进行查询（`getById`, `list`, `page` 等单表交互）。如果需要组装复杂或跨表的数据结构，应当在后端（Service/逻辑层）发起多次单表查询，并在 Java 原生层中组合数据。
- **目的**：减轻数据库计算负担、增强系统的可伸缩性与代码可维护性。

### 6.2 实体 / 文件命名去冗余
- **要求**：在建立新模块（如创建新的表、实体文件和文件夹结构）时，**不要带无关紧要的前缀**（例如 `oa_`），表名直接根据核心业务命名。
- **词汇选用**：避免使用容易撞车或语义泛指的文件夹与模块名（如通用的 `file`），请使用领域特征更明确的具体词汇（如 `documents`）。

### 6.3 数据库兼容 (适配国产化达梦 DM 数据库)
- **要求**：凡是开发了新的模块并提供建表 SQL 脚本的地方，必须在通常的 MySQL 脚本当中剥离出一份 **达梦数据库** 专用建表脚本。
- **后缀要求**：在达梦版本脚本文件名末尾增加 `-dm` 以作为区分（例如：`module_init-dm.sql`）。
- **注意差异**：达梦数据库对于 MySQL 的 `AUTO_INCREMENT` 支持不同，主键请用达梦特性 `IDENTITY(1,1)` 的等价替换。

---
> Source: [KleinHu/yiyan-hukx-backed-oa](https://github.com/KleinHu/yiyan-hukx-backed-oa) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-29 -->
