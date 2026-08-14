## base

> - You can @ files here


# Your rule content

- You can @ files here
- You can use markdown but dont have to

# 项目基础规则

## 文件结构
- @jsonHandle/ - 主项目源代码目录
- @jsonHandle Extension/ - Safari 扩展相关代码
- @jsonHandleTests/ - 单元测试目录
- @jsonHandleUITests/ - UI 测试目录

## 开发规范
- 所有代码必须遵循 Swift 标准编码规范
- 使用 SwiftLint 进行代码风格检查
- 提交代码前必须通过所有测试用例
- 新功能必须包含对应的单元测试

## 文档规范
- 所有公共 API 必须包含文档注释
- 代码变更必须更新 README.md 和 CHANGELOG.md
- 提交信息必须清晰描述变更内容

## 版本控制
- 主分支：main
- 开发分支：develop
- 功能分支：feature/*
- 修复分支：hotfix/*

## 依赖管理
- 使用 Swift Package Manager 管理依赖
- 禁止使用 CocoaPods 或 Carthage
- 所有依赖版本必须锁定在特定版本

## 安全规范
- 敏感信息必须使用环境变量或配置文件
- 用户数据必须安全存储
- 网络请求必须使用 HTTPS

---
> Source: [jojo-jie/jsonHandle](https://github.com/jojo-jie/jsonHandle) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-14 -->
