## food01

> - 编译SDK版本 (compileSdk): 35

1. Android 配置：
- 编译SDK版本 (compileSdk): 35
- 最低支持SDK版本 (minSdk): 24
- 目标SDK版本 (targetSdk): 35
- 应用包名: `com.example.food01`
- 应用版本号: 1.0 (versionCode 1)

2. Java 环境：
- Java兼容性版本: Java 17

3. Gradle 配置：
- Gradle 版本: 8.12

4. 主要依赖库：
- androidx.appcompat
- material (Material Design 组件)
- androidx.activity
- androidx.constraintlayout
- 测试相关: junit, espresso

5. 构建配置：
- 使用了新的 Version Catalog 方式管理依赖 (`libs.plugins.android.application`)
- Release 构建类型已配置，但未启用代码混淆 (`minifyEnabled false`)

项目依赖和插件要与gradle版本兼容
项目都在本地运行，不要云端
配置依赖大部分写在build.gradle文件中
所有生成代码符合规范，并生成注释

所有代码都用java写，不要使用kt，依赖包要使用java的依赖

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/02qw) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
