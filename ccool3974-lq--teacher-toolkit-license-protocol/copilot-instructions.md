## teacher-toolkit-license-protocol

> 这是 `teacher_hub` 与 `teacher_hub_license_manager` 共享的授权协议库。

# 项目名称：Teacher Toolkit License Protocol

## 项目定位

这是 `teacher_hub` 与 `teacher_hub_license_manager` 共享的授权协议库。

它负责：
- 定义授权载荷结构
- 定义协议常量
- 定义编码与解码规则
- 定义协议版本号

它不负责：
- 私钥管理
- 发码 UI
- 客户端验签页面
- 数据库存储

## 技术栈

- Dart

## 平台优先级

本项目作为纯 Dart 协议库，应尽量满足：

1. Windows
2. macOS
3. Android
4. iOS
5. 鸿蒙

## 目录约定

- `lib/src/`：协议模型、常量、编解码能力
- `lib/teacher_toolkit_license_protocol.dart`：统一导出入口
- `test/`：协议级测试

## 核心规则

- 不得在本项目中保存私钥
- 不得在本项目中实现正式签名逻辑
- 客户端和发码工具都必须依赖本项目，而不是各自复制协议结构
- 协议字段调整时，必须同步更新测试样例
- 若多个项目中的导入导出格式需要共享字段约定，例如 `操作标记`、业务主键字段、固定模板列顺序，应优先在各项目规则中统一后再实施，避免跨项目格式漂移
- 若多个项目中的导出格式需要统一，默认应统一使用 `.xlsx`
- 在 PowerShell 中读取本项目 UTF-8 文本文件时，优先使用 `Get-Content -Raw -Encoding UTF8`
- 不依赖 PowerShell 默认编码猜测来读取协议源码和文档
- 如遇中文或协议文本显示异常，应先确认文件实际 UTF-8 内容，再决定是否修改
- 若协议相关示例、文档展示或跨项目示例中涉及日期显示，默认统一使用中文年月日格式，不使用英文月份或英文日期文案

---
> Source: [ccool3974-lq/teacher_toolkit_license_protocol](https://github.com/ccool3974-lq/teacher_toolkit_license_protocol) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
