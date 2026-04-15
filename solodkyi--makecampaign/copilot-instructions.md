## makecampaign

> include: "**/*.swift"

# Cursor Rules for Donation Campaign Promo Picture App (SwiftUI, TCA)

# Include all Swift source files
include: "**/*.swift"

# Include SwiftUI previews
include: "**/*Preview*.swift"

# Include Xcode project/workspace files
include: "**/*.xcodeproj"
include: "**/*.xcworkspace"

# Include TCA-related files (Reducers, Stores, Actions, Environments)
include: "**/*Reducer*.swift"
include: "**/*Store*.swift"
include: "**/*Action*.swift"
include: "**/*Environment*.swift"

# Include resources (images, assets, etc.)
include: "**/*.xcassets"
include: "**/*.png"
include: "**/*.jpg"
include: "**/*.jpeg"
include: "**/*.svg"

# Include documentation
include: "**/*.md"
include: "**/README*"

# Include tests
include: "**/Tests/**/*.swift"
include: "**/UnitTests/**/*.swift"
include: "**/UITests/**/*.swift"

# Exclude build artifacts and derived data
exclude: "**/DerivedData/**"
exclude: "**/build/**"
exclude: "**/.build/**"
exclude: "**/.xcuserstate"
exclude: "**/*.xcuserstate"
exclude: "**/*.xcuserdata/**"
exclude: "**/*.xcuserdatad/**"

# Exclude user-specific and system files
exclude: "**/.DS_Store"
exclude: "**/.env"
exclude: "**/.git/**"
exclude: "**/.gitignore"
exclude: "**/.swiftpm/**"

# Exclude Carthage, CocoaPods, SPM artifacts
exclude: "**/Carthage/**"
exclude: "**/Pods/**"
exclude: "**/Package.resolved"

# Exclude fastlane and CI/CD config
exclude: "**/fastlane/**"
exclude: "**/.github/**"

# Exclude backup and temp files
exclude: "**/*~"
exclude: "**/#*#"
exclude: "**/.#*" 

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/solodkyi) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
