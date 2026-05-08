## ios-app-store-validator

> App Store validation for iOS projects


# iOS App Store Validator

When working with iOS project configuration files, validate against Apple's App Store Review Guidelines.

## Check Info.plist For

- Required usage description strings (NSCameraUsageDescription, NSMicrophoneUsageDescription, NSLocationWhenInUseUsageDescription, NSPhotoLibraryUsageDescription, etc.)
- User-friendly descriptions explaining WHY permission is needed
- UIRequiredDeviceCapabilities correctly declared
- CFBundleIdentifier, CFBundleVersion, CFBundleShortVersionString properly set
- No placeholder text remaining
- LSApplicationQueriesSchemes declared if querying other apps

## Check Privacy Manifest (PrivacyInfo.xcprivacy) For

- File exists if using required reason APIs
- NSPrivacyTracking accurately reflects IDFA usage
- NSPrivacyTrackingDomains lists all tracking domains
- NSPrivacyCollectedDataTypes matches actual data collection
- NSPrivacyAccessedAPITypes covers all required reason APIs

## Check Entitlements For

- Only request entitlements actually used
- Associated domains properly configured for universal links
- Special entitlements documented with explanations

## Check Swift Code For

- No private API usage
- No deprecated frameworks that cause rejection
- No competitor platform references ("Android", "Google Play")
- ATT implementation if using IDFA
- Background modes justified with actual functionality

## Output

Flag issues as:
- BLOCKER: Will cause rejection (cite guideline number)
- WARNING: May cause rejection (include risk level)

---
> Source: [gygantskiyMatilyock/ios-developer-agents](https://github.com/gygantskiyMatilyock/ios-developer-agents) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-03 -->
