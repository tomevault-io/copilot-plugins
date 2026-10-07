## aion2

> - **强制要求**：每次修改代码后，必须执行 `dotnet build` 确保没有编译错误

# Aion2 项目开发规则

## 代码修改规则

### 1. 编译检查规则
- **强制要求**：每次修改代码后，必须执行 `dotnet build` 确保没有编译错误
- **检查时机**：
  - 添加新文件后
  - 修改现有文件后
  - 添加新的NuGet包后
  - 修改项目配置后
- **错误处理**：如果发现编译错误，必须立即修复，不能继续其他开发工作
- **编译命令**：使用 `dotnet build` 或 `dotnet build aion2.csproj`

### 2. 数据库相关规则
- 修改数据库模型后，检查Entity Framework配置是否正确
- 添加新的数据库表后，更新数据库上下文和服务类
- 确保所有数据库操作都包含机器码字段
- 数据库连接失败时提供友好的错误提示

### 3. 命名规则
- 保持中文注释和变量名的一致性
- 方法名使用PascalCase
- 私有字段使用_camelCase
- 常量使用UPPER_CASE
- 数据库表名使用snake_case
- 数据库列名使用snake_case

### 4. UI开发规则
- 界面标题和按钮文本保持一致性
- 错误消息要友好且具体
- 长时间操作要显示进度或状态
- 数据库操作失败时不能影响程序正常运行

### 5. 调试规则
- 关键操作添加详细的日志记录
- 数据库操作添加异常处理
- 在DEBUG模式下显示详细调试信息
- 使用Console.WriteLine输出调试信息到控制台

### 6. 代码质量规则
- 每个公共方法都要有XML注释
- 异常处理要具体且有意义
- 避免硬编码，使用配置文件或常量
- 资源使用完毕后要正确释放（using语句）

### 7. 测试规则
- 添加新功能后要提供测试数据或测试方法
- 数据库操作要有备用方案
- 网络操作要有超时和重试机制

### 8. 文档规则
- **不需要创建功能说明文档**：完成功能后，不需要主动创建 .md 说明文档
- 代码注释要清晰完整，通过注释说明功能即可
- 只在用户明确要求时才创建文档

## 开发工作流

1. **修改代码** → 2. **编译检查** → 3. **修复错误** → 4. **功能测试** → 5. **提交代码**

## 常用命令

```bash
# 编译项目
dotnet build

# 清理并重新编译
dotnet clean && dotnet build

# 运行项目
dotnet run

# 还原NuGet包
dotnet restore
```

## 注意事项

- 中文路径可能导致PowerShell命令失败，使用绝对路径时要注意
- Entity Framework需要正确的命名空间引用：`using Microsoft.EntityFrameworkCore;`
- 异步方法要正确使用await关键字
- 数据库连接字符串中的密码要在日志中隐藏

---
> Source: [sheepGu/aion2](https://github.com/sheepGu/aion2) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-07 -->
