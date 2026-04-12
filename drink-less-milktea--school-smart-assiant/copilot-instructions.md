## school-smart-assiant

> write or test code

# 校园AI智能助手 - 测试与质量保证规范

## 测试开发规范

### 测试驱动开发(TDD)原则
**核心规则：编写任何代码之前必须先编写测试**

1. **测试优先原则**
   - 在实现任何功能代码之前，必须先编写对应的单元测试
   - 测试用例应覆盖正常流程、边界条件和异常情况
   - 确保测试失败后再开始编写实现代码
   - 实现代码必须通过所有测试用例

2. **测试编写要求**
   - 使用JUnit 4框架编写单元测试
   - 测试方法命名应清晰表达测试意图：`test[功能名称][测试场景]`
   - 每个测试方法只测试一个功能点
   - 使用断言验证预期结果
   - 添加清晰的测试注释说明测试目的

3. **测试覆盖范围**
   - MVP架构各层组件的单元测试
   - 业务逻辑的功能测试
   - 数据存储和检索的集成测试
   - UI交互的界面测试

## 质量保证标准

### 代码质量要求
1. **编译通过**：所有代码必须能够成功编译
2. **测试通过**：所有单元测试和集成测试必须通过
3. **无警告**：代码编译时不应产生警告信息
4. **代码规范**：遵循Java编码规范和项目约定

### 测试分类

#### 1. 单元测试 (Unit Tests)
- **目标**：测试单个类或方法的功能
- **工具**：JUnit 4 + Mockito
- **覆盖范围**：
  - MVP架构基础组件（BaseView、BasePresenter、BaseContract）
  - 工具类（PreferencesHelper、Constants等）
  - 数据模型类
  - Presenter业务逻辑

#### 2. 集成测试 (Integration Tests)
- **目标**：测试组件间的交互
- **工具**：AndroidJUnit4 + Espresso
- **覆盖范围**：
  - Fragment与Activity的交互
  - 数据存储和读取流程
  - 网络请求和响应处理

#### 3. UI测试 (UI Tests)
- **目标**：测试用户界面交互
- **工具**：Espresso
- **覆盖范围**：
  - 页面导航流程
  - 用户输入和响应
  - 界面状态变化

### 测试环境配置

#### 依赖配置
```gradle
dependencies {
    // 单元测试
    testImplementation 'junit:junit:4.13.2'
    testImplementation 'org.mockito:mockito-core:5.8.0'
    
    // Android测试
    androidTestImplementation 'androidx.test.ext:junit:1.2.1'
    androidTestImplementation 'androidx.test.espresso:espresso-core:3.6.1'
    androidTestImplementation 'androidx.test.espresso:espresso-contrib:3.6.1'
}
```

#### 测试运行命令
- 单元测试：`./gradlew test`
- Android测试：`./gradlew connectedAndroidTest`
- 完整构建：`./gradlew build`

### 质量检查流程

#### 开发阶段检查
1. **编写测试**：根据需求编写测试用例
2. **运行测试**：确保测试失败（红灯）
3. **实现功能**：编写最小可行代码
4. **运行测试**：确保测试通过（绿灯）
5. **重构优化**：在保持测试通过的前提下优化代码

#### 提交前检查
1. 运行所有单元测试：`./gradlew test`
2. 运行完整构建：`./gradlew build`
3. 检查代码覆盖率
4. 验证功能完整性

### 测试最佳实践

#### 1. 测试命名规范
```java
@Test
public void testPresenterAttachView() {
    // 测试Presenter附加View的功能
}

@Test
public void testViewShowErrorWithValidMessage() {
    // 测试View显示有效错误消息
}
```

#### 2. 测试结构
```java
@Test
public void testMethodName() {
    // Arrange - 准备测试数据
    
    // Act - 执行被测试的方法
    
    // Assert - 验证结果
}
```

#### 3. Mock使用
```java
@Mock
private SomeRepository mockRepository;

@Before
public void setUp() {
    MockitoAnnotations.initMocks(this);
}
```

### 性能测试

#### 1. 启动时间测试
- 应用冷启动时间 < 3秒
- 页面切换响应时间 < 1秒

#### 2. 内存使用测试
- 监控内存泄漏
- 检查大对象的生命周期

#### 3. 网络性能测试
- API响应时间监控
- 网络异常处理验证

### 错误处理测试

#### 1. 异常场景覆盖
- 网络连接失败
- 数据解析错误
- 存储空间不足
- 权限被拒绝

#### 2. 边界条件测试
- 空数据处理
- 极大数据量处理
- 特殊字符处理

### 持续集成

#### 自动化测试流程
1. 代码提交触发构建
2. 运行单元测试套件
3. 运行集成测试
4. 生成测试报告
5. 质量门禁检查

#### 质量门禁标准
- 单元测试通过率：100%
- 代码覆盖率：≥80%
- 编译警告数：0
- 关键Bug数：0

---

**版本**：v1.0  
**最后更新**：2025年1月  
**维护者**：项目开发团队

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/drink-less-milktea)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/drink-less-milktea)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
