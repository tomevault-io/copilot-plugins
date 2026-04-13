## advancedjava

> 本文档为在本代码仓库中工作的AI编码智能体提供核心操作规范。本仓库是基于**Java 17 + Maven + Spring Boot 3.3.0**的Java进阶项目，核心内容包括：Java Agent技术、并发编程模式、Java Records特性、序列化（Jackson/Fastjson2）、IO操作以及Spring框架集成。

# AGENTS.md - 代码智能体开发指南
本文档为在本代码仓库中工作的AI编码智能体提供核心操作规范。本仓库是基于**Java 17 + Maven + Spring Boot 3.3.0**的Java进阶项目，核心内容包括：Java Agent技术、并发编程模式、Java Records特性、序列化（Jackson/Fastjson2）、IO操作以及Spring框架集成。

---
## 构建命令
```bash
# 清理并编译代码
mvn clean compile
# 清理并打包（生成普通JAR + 包含所有依赖的可执行Uber JAR）
mvn clean package
# 清理并安装到本地Maven仓库
mvn clean install
# 跳过测试构建
mvn clean package -DskipTests
# 运行Spring Boot应用
mvn spring-boot:run
```

---
## 测试命令
```bash
# 运行所有测试
mvn test
# 运行单个测试类
mvn test -Dtest=CandidateProfileTest
# 运行单个测试方法
mvn test -Dtest=CandidateProfileTest#shouldTrimNameAndDefensivelyCopySkills
# 运行名称匹配指定模式的测试
mvn test -Dtest=*Record*Test
```
测试框架使用：JUnit 4.13.1 + Hamcrest断言库，测试代码统一存放在`src/test/java`目录下。

---
## 代码风格规范
### 导入顺序
1. Java标准库包（`java.*`, `javax.*`）
2. 第三方依赖库（`org.junit`, `com.alibaba.fastjson2`等）
3. 项目内部代码（`com.advancedjava.*`）
除非是教学演示必须，否则禁止随意新增第三方依赖。

### 格式化规范
- 缩进：4个空格（禁止使用Tab）
- 大括号：即使是单行语句也必须使用大括号
- 空格：关键字后加空格（`if `, `for `），运算符前后加空格（`a + b`），方法名和括号之间不加空格
- 行长度：无严格限制，优先保证可读性

### 命名规范
- 类名：大驼峰命名（`CandidateProfile`, `ConcurrencyDemoApp`）
- 方法/变量名：小驼峰命名（`hasSkill()`, `candidateProfile`）
- 测试方法名：以`should`开头或清晰描述行为（`shouldValidateNullName`）
- 包名：全小写，用点分隔（`com.advancedjava.interview.records`）

### 类型与声明规范
- 优先使用Java 17+新特性：用records作为不可变数据载体，使用密封类、模式匹配等特性
- Records规范：在紧凑构造器中完成参数校验，对可变字段做防御性拷贝
- 泛型：合理使用泛型保证类型安全，复杂类型推断场景可使用`var`关键字
- POJO规范：正确实现`equals()`, `hashCode()`, `toString()`方法，对可变参数做防御性拷贝

### 错误处理规范
- 优先使用Java标准异常（`IllegalArgumentException`, `UnsupportedOperationException`）
- 所有公共API入口必须做参数校验，包含清晰的错误信息
- 只有能显著提升代码价值时才自定义异常，编程错误优先使用非受检异常

---
## 核心约定
1. 编写代码前先参考同目录下的现有文件，保持风格一致
2. 现有文件中的中文注释要保持一致性（项目以教学演示为目的）
3. 代码清晰度优先于微优化（教学属性是本项目核心）
4. 新增功能必须包含对应的测试覆盖
5. 禁止使用Maven以外的构建工具，Java版本锁定为17+
6. Spring组件开发：遵循Spring Boot规范，优先使用构造注入而非字段注入

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Chinazhouwy)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Chinazhouwy)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
