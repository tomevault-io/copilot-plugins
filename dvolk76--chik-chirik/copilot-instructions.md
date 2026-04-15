## chik-chirik

> iOS 17 SwiftUI / MVVM project rules (без тестов и Xcode Cloud)


# 📐 Архитектура

1. Проект строится на **MVVM**.  
2. View-модели наследуют `ObservableObject`; импорт `SwiftUI` в них запрещён.  
3. Навигация — `NavigationStack`; переходы описываются в родительской ViewModel.

# 📁 Структура каталогов
Sources/
 ├─ App/                 // AppEntry, AppState
 ├─ Core/                // Модели, сервисные протоколы
 ├─ Features/            // <Feature>/View + ViewModel
 ├─ UIComponents/        // Общие SwiftUI-компоненты
 └─ Resources/           // Assets, Localizable.strings

# 🔌 Dependency Injection
* Протокол `AppContainer` — единая точка доступа к сервисам.  
* Внедрение через инициализаторы или `@Environment(\.di)`.

# ☁️ Firebase
SDK вызывается только через сервис-обёртки (`AuthService`, `DBService` и пр.);  
ViewModel видит лишь их протоколы.

# 📸 QR-коды + Universal Links
* Генерация — `CIFilter.qrCodeGenerator`;  
* Сид-фраза — в Keychain (`kSecAttrAccessibleAfterFirstUnlock`);  
* Link: `https://example.com/link/<payload>`.

# 🧹 Код-стайл
SwiftLint обязателен (см. .swiftlint.yml)  
Запрещены `print()` и `TODO:`.  
Строка ≤ 120 симв., cyclomatic ≤ 15/25 (warn/error).

# 🗂 Документация
Публичные API оформляются Markup-комментарием, язык един внутри файла.

# 🛡 Security & Perf
* Использовать SecureEnclave/Keychain там, где храним сид-фразу.  
* Избегать блокирующих операций в `@MainActor`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Dvolk76)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Dvolk76)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
