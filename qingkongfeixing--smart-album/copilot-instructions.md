## smart-album

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 项目概览

Flutter 本地相册应用（随搜相册）。通过云端视觉大模型（兼容 OpenAI/Claude API）为图片生成标签，支持本地关键词搜索。SDK ^3.12.0，Material 3，Provider 状态管理。

## 构建 & 检查

```bash
flutter analyze lib/screens/gallery_screen.dart  # 单文件检查
flutter analyze                                   # 全项目检查
flutter build apk --release                       # 构建发布 APK
```

输出 APK：`build/app/outputs/flutter-apk/app-release.apk`

## 核心架构

```
lib/
  main.dart              # 入口，Provider 注入，主题配置
  models/
    photo.dart           # Photo 实体 + SearchResult 类
    database_helper.dart # sqflite 单例，photo 表 CRUD
  services/
    photo_scanner.dart   # 扫描相册（MethodChannel→Android MediaStore），批量/单张云端解析
    cloud_enhance.dart   # 多模型云端服务：最多5个模型并行，图片压缩→base64→API，排除文件夹管理
    notification_service.dart
  screens/
    gallery_screen.dart  # 主界面：文件夹网格→照片网格（日期分组），多选/复制/剪切/粘贴/删除，快速打标签
    search_screen.dart   # 本地关键词搜索
    settings_screen.dart # 云端模型配置，扫描触发，排除文件夹管理
  widgets/
    photo_detail.dart    # 大图查看 + PageView 左右翻页
  utils/
    constants.dart       # AppConstants（数据库名、默认 API 地址等）
    permissions.dart     # 存储权限请求
```

## 数据流

1. `PhotoScanner` 通过 Android MethodChannel 读取 MediaStore，去重后写入 `photos` 表
2. `CloudEnhanceService` 将图片压缩为 JPEG base64，调用兼容 OpenAI 的 `/chat/completions` 视觉接口，返回标签写入 `tags` 字段
3. `GalleryScreen` 按文件夹路径聚合 → 按日期分组展示，通过 `Provider` 获取 `PhotoScanner` 和 `CloudEnhanceService` 实例
4. 搜索扫描 `tags` 和 `ocr_text` 字段做关键词匹配

## 关键状态

- `GalleryScreen._selectedFolder`：`null`→文件夹网格主页，非 `null`→照片网格详情，`AnimatedController` 控制缩放进出动画
- `GalleryScreen._selectMode` / `_folderSelectMode`：照片多选 / 文件夹多选模式
- `GalleryScreen._cbPhotos` / `_cbIsCut`：剪贴板（复制/剪切→粘贴到目标文件夹）
- `PhotoScanner.state`（`ValueNotifier<ScanState>`）：idle/scanning/completed/error
- `PhotoScanner.onProgress`（`ValueNotifier<ScanProgress?>`）：扫描进度

---
> Source: [qingkongfeixing/smart-album](https://github.com/qingkongfeixing/smart-album) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-02 -->
