## cmux-mobile

> 发布到 App Store 前必须完成以下检查：

# cmux-mobile agent notes

## Release Checklist

发布到 App Store 前必须完成以下检查：

### Push Notifications（当前未启用）

Push 功能需要以下步骤才能启用：

1. 登录 Apple Developer Portal → Certificates, Identifiers & Profiles → Identifiers
2. 找到 App ID `com.rooyun.devpod`，勾选 Push Notifications capability
3. 在 `SupportingFiles/cmux-mobile.entitlements` 中添加：
   ```xml
   <key>aps-environment</key>
   <string>development</string>
   ```
4. 发布时将 `development` 改为 `production`
5. Xcode 中重新下载 provisioning profile（自动签名会自动处理）

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/no1coder) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
