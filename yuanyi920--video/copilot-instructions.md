## video

> 爬取91porn网站视频并播放的Android应用。

# 91视频 Flutter 项目

## 项目概述
爬取91porn网站视频并播放的Android应用。

## 关键命令
- `flutter analyze` - 仅做代码分析，不执行构建（必须通过）
- `flutter run -d <device_id>` - 运行应用
- `flutter build apk` - 构建APK

## 项目结构
- `lib/config/` - 配置文件（API地址、应用设置）
- `lib/models/` - 数据模型（VideoModel、VideoDetailModel、CommentModel）
- `lib/pages/` - 页面（VideoListPage、VideoDetailPage）
- `lib/services/` - API服务（HTML解析、网络请求）

## 核心依赖
- `dio` - HTTP请求
- `html` - HTML解析（用于爬虫）
- `better_player_plus` - 视频播放
- `screen_brightness` - 屏幕亮度控制
- `wakelock_plus` - 播放时保持屏幕常亮

## 技术要点
- 视频URL通过JavaScript动态生成，需用正则从`strencode2`函数提取并解码
- API分类参数在`ApiConfig.categories`中定义
- TabBar实现视频分类切换

## 代码规范
- 使用`flutter_lints`进行代码检查
- 分析排除：build、android、ios等目录

---
> Source: [yuanyi920/video](https://github.com/yuanyi920/video) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-31 -->
