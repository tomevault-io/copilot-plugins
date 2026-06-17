## jimeng-digital-human-skill

> 调用火山引擎即梦AI的OmniHuman1.5数字人API，根据用户提供的图片和音频生成数字人说话视频。Use when the user asks to generate digital human video, create talking avatar, 生成数字人视频, 数字人, 即梦, OmniHuman, or 让图片说话.


# 即梦数字人 OmniHuman1.5 技能

## 技能描述

调用火山引擎即梦AI的OmniHuman1.5数字人API，根据用户提供的图片和音频生成数字人说话视频。

## 触发词

- "生成数字人视频"
- "数字人"
- "即梦"
- "OmniHuman"
- "让图片说话"

## 使用方式

用户提供图片URL和音频URL，技能自动调用API生成视频并返回下载链接。

## 参数说明

| 参数 | 必选 | 说明 |
|------|------|------|
| image_url | 是 | 人像图片URL，公网可访问，支持人物/宠物/动漫等 |
| audio_url | 是 | 音频URL，时长≤35秒，公网可访问 |
| prompt | 否 | 提示词（中文/英文/日文/韩文），控制画面动作 |

## 环境要求

需要配置以下环境变量（在 .env 文件中）：

```bash
VOLCENGINE_ACCESS_KEY_ID=你的AK
VOLCENGINE_SECRET_ACCESS_KEY=你的SK
```

获取密钥：https://console.volcengine.com/iam/keymanage

## 前置条件

1. 火山引擎账号并完成实名认证
2. 开通 OmniHuman1.5 服务：https://console.volcengine.com/ai/overview
3. 创建 AK/SK 密钥

## 输出

返回生成的视频下载链接（有效期1小时）。

## 注意事项

- 音频时长不能超过35秒
- 图片需包含清晰的人物正面
- 视频生成约需60-120秒
- 视频链接有效期1小时

## 常见错误

| 错误码 | 说明 | 解决方案 |
|--------|------|----------|
| 50400 | 服务未开通 | 前往控制台开通 OmniHuman1.5 |
| 50215 | 音频时长超限 | 裁剪音频至35秒以内 |
| 50411 | 图片审核未通过 | 更换合规图片 |

## 相关链接

- API文档：https://www.volcengine.com/docs/85621/1829013
- 控制台：https://console.volcengine.com/ai/overview
- 密钥管理：https://console.volcengine.com/iam/keymanage

---
> Source: [78design/jimeng-digital-human-skill](https://github.com/78design/jimeng-digital-human-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
