## xclone-ddd

> description: UUID型IDの実装パターン

---
description: UUID型IDの実装パターン
globs: **/domain/model/**/*Id.kt
alwaysApply: false
---
# UUID型IDの実装パターン

## 実装例
```kotlin
@JvmInline
value class UserId private constructor(val value: UUID) {
    companion object {
        fun generate(): UserId = UserId(UUID.randomUUID())
        
        fun from(value: String): UserId {
            try {
                return UserId(UUID.fromString(value))
            } catch (e: IllegalArgumentException) {
                throw IllegalArgumentException("不正なID形式です: $value", e)
            }
        }
    }
    
    override fun toString(): String = value.toString()
}
```

## パターン
- 値オブジェクトとしてのID型
- 型安全性の確保
- 文字列/UUID相互変換
- 生成メソッドの提供 

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/small-java-world) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
