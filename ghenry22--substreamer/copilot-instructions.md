## native-modules

> How to create and register local Expo native modules in the modules/ directory


# Local Expo Native Modules

Custom native functionality lives in `modules/` as local Expo modules. Current modules: `expo-async-fs`, `expo-ssl-trust`, `expo-gzip`, `expo-backup-exclusions`, `expo-move-to-back`, `expo-image-resize`, `expo-image-colors`, a local fork of `react-native-track-player`, and a local fork of `subsonic-api`. Follow existing modules as reference.

## Directory Structure

```
modules/{module-name}/
  android/
    build.gradle                                  # REQUIRED — Gradle won't compile the module without it
    src/main/java/expo/modules/{moduleid}/
      {ModuleName}Module.kt
  ios/
    {ModuleName}.podspec
    {ModuleName}Module.swift
  src/
    {ModuleName}Module.ts    # requireNativeModule + fallback stub
    index.ts                 # Public typed exports
  expo-module.config.json
  package.json
```

## Registration Checklist

A local module requires **four** registration steps or Metro/native builds will silently leave the module out of the APK (the JS wrapper will hit its fallback stub and log "Native module not found"):

1. **`expo-module.config.json`** -- declares platform module classes:
```json
{
  "platforms": ["ios", "android"],
  "ios": { "modules": ["{ModuleName}Module"] },
  "android": { "modules": ["expo.modules.{moduleid}.{ModuleName}Module"] }
}
```

2. **`package.json`** -- standard npm package metadata with `"main": "src/index.ts"`:
```json
{
  "name": "{module-name}",
  "version": "1.0.0",
  "main": "src/index.ts",
  "types": "src/index.ts",
  "peerDependencies": { "expo": "*", "react": "*", "react-native": "*" }
}
```

3. **`android/build.gradle`** -- **critical**: without this the Android autolinker finds the module via `expo-module.config.json` but never compiles it into the APK. Follow the canonical template (see `modules/expo-async-fs/android/build.gradle`):
```gradle
apply plugin: 'com.android.library'
apply plugin: 'kotlin-android'

group = 'expo.modules.{moduleid}'
version = '1.0.0'

def expoModulesCorePlugin = new File(project(":expo-modules-core").projectDir.absolutePath, "ExpoModulesCorePlugin.gradle")
if (expoModulesCorePlugin.exists()) {
  apply from: expoModulesCorePlugin
  applyKotlinExpoModulesCorePlugin()
}

android {
  namespace "expo.modules.{moduleid}"
  compileSdk rootProject.ext.has("compileSdkVersion") ? rootProject.ext.get("compileSdkVersion") : 35
  defaultConfig {
    minSdk rootProject.ext.has("minSdkVersion") ? rootProject.ext.get("minSdkVersion") : 24
    targetSdk rootProject.ext.has("targetSdkVersion") ? rootProject.ext.get("targetSdkVersion") : 35
  }
  compileOptions {
    sourceCompatibility JavaVersion.VERSION_17
    targetCompatibility JavaVersion.VERSION_17
  }
  kotlinOptions {
    jvmTarget = JavaVersion.VERSION_17.majorVersion
  }
}

dependencies {
  implementation project(':expo-modules-core')
  implementation "com.facebook.react:react-android"
}
```

4. **Root `package.json`** -- **critical**: add a dependency entry, a doctor exclusion, **and** an install exclusion (prevents `expo install` from overwriting the local version):
```json
{
  "dependencies": {
    "{module-name}": "file:./modules/{module-name}"
  },
  "expo": {
    "install": {
      "exclude": ["{module-name}"]
    },
    "doctor": {
      "reactNativeDirectoryCheck": {
        "exclude": ["{module-name}"]
      }
    }
  }
}
```
Then run `npm install` to create the symlink in `node_modules/`.

## Native Code Pattern

Use `AsyncFunction` for background-thread work (the Expo Modules API handles thread dispatch automatically):

```swift
// iOS
AsyncFunction("myFunction") { (arg: String) -> String in
  // Runs on native background thread
  return result
}
```

```kotlin
// Android
AsyncFunction("myFunction") { arg: String ->
  // Runs on native background thread
  result
}
```

## JS Wrapper Pattern

Always provide a graceful fallback when the native module isn't available (e.g. during JS-only development):

```typescript
// src/{ModuleName}Module.ts
import { requireNativeModule } from 'expo-modules-core';

let module: NativeInterface;
try {
  module = requireNativeModule('{ModuleName}');
} catch {
  console.warn('[{module-name}] Native module not found. Rebuild the app.');
  module = { /* no-op stubs */ } as any;
}
export default module;
```

## After Creating a Module

A **native rebuild** is required (`npx expo run:ios` / `npx expo run:android`). Metro bundling alone is not sufficient for new native modules.

---
> Source: [ghenry22/substreamer](https://github.com/ghenry22/substreamer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-22 -->
