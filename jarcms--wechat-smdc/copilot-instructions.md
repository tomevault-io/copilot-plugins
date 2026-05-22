## wechat-smdc

> - 服务端：JAVA、Spring Boot、Spring Framework、Maven、Mybatis-Plus、以及其他JAVA相关技术

## 角色定义

你是一个经验丰富的开发者，擅长技术：
- 服务端：JAVA、Spring Boot、Spring Framework、Maven、Mybatis-Plus、以及其他JAVA相关技术
- 前端：TypeScript、Node.js、Element UI等其他前端组件
- 微信小程序：微信组件、微信支付、微信登录等微信相关技术

## 项目结构
根目录/
├── src/                                # Java 后端源代码
│   ├── main/
│   │   ├── java/com/shechubbb/smdc/
│   │   │   ├── config/                 # 配置类
│   │   │   ├── controller/             # 控制器
│   │   │   │   ├── admin/              # 管理后台控制器，Controller统一以Admin作为前缀
│   │   │   │   └── mini/               # 小程序控制器
│   │   │   ├── entity/                 # 实体类
│   │   │   ├── mapper/                 # Mybatis-Plus Mapper接口
│   │   │   ├── service/                # 服务接口
│   │   │   │   └── impl/               # 服务实现类
│   │   │   ├── vo/                     # 视图对象
│   │   │   ├── common/                 # 公共组件
│   │   │   │   ├── exception/          # 异常处理
│   │   │   │   ├── util/               # 工具类
│   │   │   │   └── result/             # 统一响应结果
│   │   │   └── *Application.java       # 应用启动类
│   │   ├── resources/
│   │   │   ├── application.yml         # 应用配置文件
├── miniapp/                            # 微信小程序
├── admin/                              # Vue管理后台
├── .gitignore                          # Git忽略文件
├── pom.xml                             # Maven配置
└── README.md                           # 项目说明


## 强制规定

1. **一致性**：有任何更改，必须保证服务端（src目录）、微信小程序（miniapp目录）、管理后台（admin目录）的一致性
2. **代码清理**：替换新的解决方案后，要检查之前的代码是否还有使用，没有用就删除掉
3. **错误检查**：每次代码修改后，必须检查编译器/开发工具是否报告错误并立即解决
4. **零错误提交**：禁止提交包含编译错误的代码到代码库
5. **数据库更新一致性**：数据库实体定义有变更时，必须更新相关的初始化SQL，并提供更新SQL

## 开发规范

### 技术栈规范

#### 服务端
- **核心框架**：Spring Boot
- **ORM框架**：Mybatis-Plus
- **数据库**：MySQL
- **构建工具**：Maven
- **JDK版本**：JDK8

#### 微信小程序
- **基础技术**：微信小程序原生技术
- **UI组件**：Element UI等第三方组件

#### 管理后台
- **前端框架**：Vue
- **UI组件库**：Element UI、Element Plus
- **扩展组件**：其他第三方组件

### 接口规范

- **交互协议**：服务端和前端交互协议为POST，请求参数为JSON，返回数据为JSON
- **响应格式**：统一响应格式：`{"code":1,"msg":"errorMsg","data":obj}`
- **成功状态**：`code=1` 代表成功，`data`为具体的数据
- **失败状态**：`code=0` 代表失败，`msg`字段为错误信息用于提示

### 微信小程序开发规范

#### 颜色管理
- **统一定义**：所有颜色值必须在 `miniapp/styles/theme.wxss` 中统一定义
- **禁止硬编码**：禁止在页面或组件中直接使用色值（如 #FFFFFF）
- **变量引用**：使用CSS变量方式引用颜色值

#### 资源使用
- **图标优先级**：优先使用微信小程序内置图标
- **替代方案**：如无合适内置图标，可安装其他图标库依赖
- **最后选择**：最后考虑使用图片资源

### 后端已实现接口列表

#### 用户相关接口
##### 小程序端接口
- **POST /mini/user/login** - 微信登录
- **POST /mini/user/account/login** - 账号密码登录
- **POST /mini/user/info** - 获取当前用户信息
- **GET /mini/user/info/{id}** - 根据ID获取用户信息（管理员权限）
- **POST /mini/user/update** - 更新用户信息

##### 管理端接口
- **POST /admin/employee/login** - 员工登录
- **GET /admin/employee/info/{id}** - 获取员工信息
- **GET /admin/employee/page** - 员工列表分页查询
- **POST /admin/employee/add** - 添加员工
- **POST /admin/employee/update** - 更新员工信息

#### 分类相关接口
##### 小程序端接口
- **GET /mini/category/list** - 获取分类列表

##### 管理端接口
- **GET /admin/category/page** - 分类分页查询
- **GET /admin/category/list** - 分类列表查询
- **POST /admin/category/add** - 添加分类
- **POST /admin/category/update** - 更新分类
- **POST /admin/category/delete** - 删除分类

#### 菜品相关接口
##### 小程序端接口
- **GET /mini/dish/list/{categoryId}** - 根据分类ID获取菜品列表
- **GET /mini/dish/list?categoryId=** - 根据分类ID获取菜品列表（请求参数方式）
- **GET /mini/dish/detail/{id}** - 获取菜品详情

##### 管理端接口
- **GET /admin/dish/page** - 菜品分页查询
- **GET /admin/dish/info/{id}** - 获取菜品信息
- **POST /admin/dish/add** - 添加菜品
- **POST /admin/dish/update** - 更新菜品
- **POST /admin/dish/delete** - 删除菜品

#### 桌位相关接口
##### 管理端接口
- **GET /admin/table/page** - 桌位分页查询
- **POST /admin/table/add** - 添加桌位
- **POST /admin/table/update** - 更新桌位
- **POST /admin/table/delete** - 删除桌位
- **GET /admin/table/qrcode/{id}** - 生成桌位二维码

#### 订单相关接口
##### 小程序端接口
- **POST /mini/order/create** - 创建订单
- **POST /mini/order/pay** - 支付订单
- **POST /mini/order/cancel** - 取消订单
- **GET /mini/order/list** - 订单列表
- **GET /mini/order/detail/{id}** - 订单详情
- **GET /mini/order/table/{code}** - 扫码获取桌位信息

##### 管理端接口
- **GET /admin/order/page** - 订单分页查询
- **GET /admin/order/detail/{id}** - 获取订单详情
- **POST /admin/order/accept** - 接单
- **POST /admin/order/complete** - 完成订单
- **POST /admin/order/cancel** - 取消订单

#### 店铺信息相关接口
##### 小程序端接口
- **POST /mini/shop/info** - 获取店铺信息

##### 管理端接口
- **GET /admin/shop/info** - 获取店铺信息
- **POST /admin/shop/update** - 更新店铺信息


### 小程序端已实现页面

#### 基础页面
- **登录页面** (`pages/login/login`) - 用户登录页面，支持微信登录和账号密码登录
- **首页** (`pages/index/index`) - 应用首页，展示店铺信息和热门菜品
- **分类页** (`pages/category/category`) - 展示菜品分类列表
- **菜品页** (`pages/dish/dish`) - 展示具体分类下的菜品列表
- **购物车** (`pages/cart/cart`) - 用户已选菜品的购物车

#### 订单相关页面
- **订单列表** (`pages/order/list/list`) - 用户的历史订单列表
- **订单确认** (`pages/order/confirm/confirm`) - 确认订单信息和支付
- **订单详情** (`pages/order/detail/detail`) - 查看订单的详细信息

#### 用户相关页面
- **用户中心** (`pages/user/user`) - 用户个人中心，展示用户信息和功能入口

#### 导航结构
- **底部Tab导航**：主页和用户中心
- **顶部导航**：页面标题"扫码点餐"，采用品牌主色调 #FF5722


### 数据库表定义
-- 用户表
CREATE TABLE IF NOT EXISTS `user` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT '主键',
  `open_id` varchar(32) COLLATE utf8mb4_unicode_ci DEFAULT NULL COMMENT '微信openid',
  `session_key` varchar(255) DEFAULT NULL COMMENT '会话密钥',
  `nick_name` varchar(32) COLLATE utf8mb4_unicode_ci DEFAULT NULL COMMENT '昵称',
  `username` varchar(50) DEFAULT NULL COMMENT '用户名',
  `password` varchar(255) DEFAULT NULL COMMENT '密码',
  `avatar_url` varchar(255) COLLATE utf8mb4_unicode_ci DEFAULT NULL COMMENT '头像',
  `gender` tinyint(2) DEFAULT '0' COMMENT '性别 0-未知 1-男 2-女',
  `city` varchar(32) COLLATE utf8mb4_unicode_ci DEFAULT NULL COMMENT '城市',
  `province` varchar(32) COLLATE utf8mb4_unicode_ci DEFAULT NULL COMMENT '省份',
  `country` varchar(32) COLLATE utf8mb4_unicode_ci DEFAULT NULL COMMENT '国家',
  `phone` varchar(11) COLLATE utf8mb4_unicode_ci DEFAULT NULL COMMENT '手机号',
  `last_login_time` datetime DEFAULT NULL COMMENT '最后登录时间',
  `create_time` datetime DEFAULT NULL COMMENT '创建时间',
  `update_time` datetime DEFAULT NULL COMMENT '更新时间',
  PRIMARY KEY (`id`) USING BTREE
) ENGINE=InnoDB AUTO_INCREMENT=10000 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci COMMENT='用户';

-- 添加测试用户
INSERT INTO `user` (`id`, `nick_name`, `avatar_url`, `gender`, `city`, `province`, `country`, `phone`, `session_key`, `username`, `password`, `last_login_time`, `create_time`, `update_time`) 
VALUES (9999, '测试用户', '/images/default-avatar.png', 1, '广州', '广东', '中国', '13800138000', 'init_session_key', 'test', '123456', NOW(), NOW(), NOW());

-- 员工表
CREATE TABLE IF NOT EXISTS `employee` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT '主键',
  `username` varchar(32) COLLATE utf8mb4_unicode_ci NOT NULL COMMENT '用户名',
  `password` varchar(64) COLLATE utf8mb4_unicode_ci NOT NULL COMMENT '密码',
  `name` varchar(32) COLLATE utf8mb4_unicode_ci NOT NULL COMMENT '姓名',
  `phone` varchar(11) COLLATE utf8mb4_unicode_ci DEFAULT NULL COMMENT '手机号',
  `role` tinyint(4) NOT NULL DEFAULT '2' COMMENT '角色，1管理员，2普通员工',
  `create_time` datetime DEFAULT NULL COMMENT '创建时间',
  `update_time` datetime DEFAULT NULL COMMENT '更新时间',
  PRIMARY KEY (`id`),
  UNIQUE KEY `idx_username` (`username`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci COMMENT='员工信息表';

-- 初始化管理员账号，密码为123456的MD5值
INSERT INTO `employee` (`id`, `username`, `password`, `name`, `role`, `create_time`, `update_time`) 
VALUES (1, 'admin', 'e10adc3949ba59abbe56e057f20f883e', '管理员', 1, NOW(), NOW());

-- 添加测试账号，密码为123456的MD5值
INSERT INTO `employee` (`id`, `username`, `password`, `name`, `role`, `phone`, `create_time`, `update_time`) 
VALUES (2, 'test', 'e10adc3949ba59abbe56e057f20f883e', '测试账号', 2, '13800138000', NOW(), NOW());

-- 分类表
CREATE TABLE IF NOT EXISTS `category` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT '主键',
  `name` varchar(32) COLLATE utf8mb4_unicode_ci NOT NULL COMMENT '分类名称',
  `sort` int(11) NOT NULL DEFAULT '0' COMMENT '排序号',
  `create_time` datetime DEFAULT NULL COMMENT '创建时间',
  `update_time` datetime DEFAULT NULL COMMENT '更新时间',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci COMMENT='菜品分类表';

-- 菜品表
CREATE TABLE IF NOT EXISTS `dish` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT '主键',
  `name` varchar(32) COLLATE utf8mb4_unicode_ci NOT NULL COMMENT '菜品名称',
  `category_id` bigint(20) NOT NULL COMMENT '分类id',
  `price` decimal(10,2) NOT NULL COMMENT '价格',
  `image` varchar(255) COLLATE utf8mb4_unicode_ci DEFAULT NULL COMMENT '图片',
  `description` varchar(255) COLLATE utf8mb4_unicode_ci DEFAULT NULL COMMENT '描述信息',
  `status` tinyint(4) NOT NULL DEFAULT '1' COMMENT '售卖状态 0:停售 1:起售',
  `sort` int(11) NOT NULL DEFAULT '0' COMMENT '排序号',
  `create_time` datetime DEFAULT NULL COMMENT '创建时间',
  `update_time` datetime DEFAULT NULL COMMENT '更新时间',
  PRIMARY KEY (`id`),
  KEY `idx_category_id` (`category_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci COMMENT='菜品表';

-- 规格表
CREATE TABLE IF NOT EXISTS `specification` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT '主键',
  `dish_id` bigint(20) NOT NULL COMMENT '菜品id',
  `name` varchar(32) COLLATE utf8mb4_unicode_ci NOT NULL COMMENT '规格名称',
  `price` decimal(10,2) NOT NULL COMMENT '价格',
  `create_time` datetime DEFAULT NULL COMMENT '创建时间',
  `update_time` datetime DEFAULT NULL COMMENT '更新时间',
  PRIMARY KEY (`id`),
  KEY `idx_dish_id` (`dish_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci COMMENT='规格表';

-- 桌位表
CREATE TABLE IF NOT EXISTS `table_info` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT '主键',
  `name` varchar(32) COLLATE utf8mb4_unicode_ci NOT NULL COMMENT '桌位名称',
  `code` varchar(32) COLLATE utf8mb4_unicode_ci NOT NULL COMMENT '桌位二维码',
  `status` tinyint(4) NOT NULL DEFAULT '0' COMMENT '状态 0:空闲 1:使用中',
  `capacity` int(11) DEFAULT NULL COMMENT '容纳人数',
  `create_time` datetime DEFAULT NULL COMMENT '创建时间',
  `update_time` datetime DEFAULT NULL COMMENT '更新时间',
  PRIMARY KEY (`id`),
  UNIQUE KEY `idx_code` (`code`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci COMMENT='桌位表';

-- 订单表
CREATE TABLE IF NOT EXISTS `order` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT '主键',
  `number` varchar(50) COLLATE utf8mb4_unicode_ci NOT NULL COMMENT '订单号',
  `user_id` bigint(20) NOT NULL COMMENT '用户id',
  `table_id` bigint(20) NOT NULL COMMENT '桌位id',
  `amount` decimal(10,2) NOT NULL COMMENT '总金额',
  `status` tinyint(4) NOT NULL DEFAULT '1' COMMENT '订单状态 1:待付款 2:待接单 3:待上菜 4:已完成 5:已取消',
  `pay_method` tinyint(4) DEFAULT NULL COMMENT '支付方式 1:微信支付 2:支付宝支付',
  `pay_status` tinyint(4) NOT NULL DEFAULT '0' COMMENT '支付状态 0:未支付 1:已支付',
  `remark` varchar(255) COLLATE utf8mb4_unicode_ci DEFAULT NULL COMMENT '备注',
  `create_time` datetime DEFAULT NULL COMMENT '创建时间',
  `update_time` datetime DEFAULT NULL COMMENT '更新时间',
  PRIMARY KEY (`id`),
  UNIQUE KEY `idx_number` (`number`),
  KEY `idx_user_id` (`user_id`),
  KEY `idx_table_id` (`table_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci COMMENT='订单表';

-- 订单明细表
CREATE TABLE IF NOT EXISTS `order_detail` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT '主键',
  `order_id` bigint(20) NOT NULL COMMENT '订单id',
  `dish_id` bigint(20) NOT NULL COMMENT '菜品id',
  `specification_id` bigint(20) DEFAULT NULL COMMENT '规格id',
  `number` int(11) NOT NULL COMMENT '数量',
  `amount` decimal(10,2) NOT NULL COMMENT '金额',
  `create_time` datetime DEFAULT NULL COMMENT '创建时间',
  PRIMARY KEY (`id`),
  KEY `idx_order_id` (`order_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci COMMENT='订单明细表';

-- 店铺信息表
CREATE TABLE `shop_info` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT '主键',
  `name` varchar(50) COLLATE utf8mb4_unicode_ci NOT NULL COMMENT '店铺名称',
  `slogan` varchar(100) COLLATE utf8mb4_unicode_ci DEFAULT NULL COMMENT '店铺标语',
  `logo` varchar(255) COLLATE utf8mb4_unicode_ci DEFAULT NULL COMMENT '店铺logo',
  `address` varchar(255) COLLATE utf8mb4_unicode_ci DEFAULT NULL COMMENT '店铺地址',
  `phone` varchar(20) COLLATE utf8mb4_unicode_ci DEFAULT NULL COMMENT '联系电话',
  `business_hours` varchar(50) COLLATE utf8mb4_unicode_ci DEFAULT NULL COMMENT '营业时间',
  `longitude` varchar(20) COLLATE utf8mb4_unicode_ci DEFAULT NULL COMMENT '经度',
  `latitude` varchar(20) COLLATE utf8mb4_unicode_ci DEFAULT NULL COMMENT '纬度',
  `status` tinyint(4) NOT NULL DEFAULT '1' COMMENT '状态，0关闭，1营业中',
  `create_time` datetime DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
  `update_time` datetime DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新时间',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci COMMENT='店铺信息表';


### JAVA代码规范

#### 代码风格与结构
- **代码质量**：编写清晰、高效且有良好文档的Java代码
- **最佳实践**：遵循Spring Boot最佳实践和约定
- **API设计**：实现RESTful API设计模式
- **命名规范**：使用描述性方法和变量名，遵循驼峰命名法
- **项目结构**：结构化Spring Boot应用：控制器、服务、仓库、模型、配置

#### Spring Boot 规范
- **依赖管理**：使用Spring Boot starters快速设置项目和管理依赖
- **注解使用**：正确使用注解（如@SpringBootApplication、@RestController、@Service）
- **自动配置**：有效利用Spring Boot的自动配置功能
- **异常处理**：使用@ControllerAdvice和@ExceptionHandler实现异常处理

#### 命名约定
- **类名**：使用帕斯卡命名法（如UserController、OrderService）
- **方法和变量**：使用驼峰命名法（如findUserById、isOrderValid）
- **常量**：使用全大写（如MAX_RETRY_ATTEMPTS、DEFAULT_PAGE_SIZE）

#### Java和Spring Boot使用
- **Java 8特性**：适当使用Java 8特性（如Lambda表达式、流API、Optional、新的日期/时间API）
- **版本特性**：使用Spring Boot 2.6.13特性和最佳实践
- **数据访问**：使用Spring Data JPA进行数据库操作
- **数据验证**：使用Bean Validation实现适当的验证（如@Valid、自定义验证器）

#### 配置与属性
- **配置文件**：使用application.yml进行配置
- **环境配置**：使用Spring Profiles实现环境特定配置
- **类型安全**：使用@ConfigurationProperties实现类型安全的配置属性

#### 依赖注入与IoC
- **注入方式**：使用构造函数注入而非字段注入，提高可测试性
- **容器管理**：利用Spring的IoC容器管理Bean生命周期

#### 性能与可扩展性
- **缓存策略**：使用Spring Cache抽象实现缓存策略，使用Guava作为缓存解决方案
- **异步处理**：使用@Async进行非阻塞操作
- **查询优化**：实现适当的数据库索引和查询优化

#### 安全
- **身份验证**：使用Spring Security进行身份验证和授权
- **密码安全**：使用适当的密码编码（如BCrypt）
- **跨域设置**：需要时实现CORS配置

#### 日志与监控
- **日志框架**：使用SLF4J和Logback进行日志记录
- **日志级别**：实现适当的日志级别（ERROR、WARN、INFO、DEBUG）
- **应用监控**：使用Spring Boot Actuator进行应用监控和指标收集

#### 数据访问与ORM
- **数据操作**：使用Mybatis-plus进行数据库操作
- **关系管理**：不允许使用外键等数据库关联放回寺

#### 构建与部署
- **依赖构建**：使用Maven进行依赖管理和构建过程
- **配置简化**：使用单环境application.yml
- **容器化**：适用时使用Docker进行容器化

#### 遵循最佳实践
- **API设计**：RESTful API设计（正确使用HTTP方法、状态码等）
- **架构选择**：微服务架构（如适用）
- **异步编程**：使用Spring的@Async或Spring WebFlux的响应式编程进行异步处理

**核心原则**：遵循SOLID原则，在Spring Boot应用设计中保持高内聚和低耦合。

---
> Source: [jarcms/wechat-smdc](https://github.com/jarcms/wechat-smdc) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
