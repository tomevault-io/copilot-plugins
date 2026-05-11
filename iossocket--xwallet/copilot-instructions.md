## xwallet

> iOS-native multi-chain crypto wallet built with SwiftUI + TCA (The Composable Architecture).

# CLAUDE.md — xWallet

## Project Summary

iOS-native multi-chain crypto wallet built with SwiftUI + TCA (The Composable Architecture).
Target chains: EVM (Ethereum + L2s) and Starknet. Privacy layer (Railgun/Zcash) planned.
Minimum deployment: iOS 16.4 / Xcode 16.4 / Swift 5.9.

## Module Map

```
xWallet/                          # App target
├── xWalletApp.swift              # @main entry — creates root Store<AppFeature>
├── Core/
│   └── WalletCoreValidator.swift # WalletCore sanity check (currently disabled)
├── Features/
│   ├── App/
│   │   ├── AppFeature.swift      # Root @Reducer — composes Account, Wallet, Settings
│   │   ├── ContentView.swift     # Main content view
│   │   └── RootView.swift        # Launch-phase routing (booting → onboarding → ready)
│   ├── Account/
│   │   └── Account.swift         # @Reducer — wallet creation, mnemonic/PK import, onboarding flow
│   ├── Wallet/
│   │   └── Wallet.swift          # @Reducer — balance fetch, asset list, receive sheet
│   ├── Send/
│   │   └── Send.swift            # @Reducer — EVM send tx (WIP, body incomplete)
│   └── Settings/
│       └── Settings.swift        # @Reducer — RPC URL config, connection check
├── Models/
│   └── AssetItem.swift           # Asset display model
├── Services/
│   ├── WalletClient.swift        # DependencyKey — multi-wallet CRUD, SQLite (GRDB) + Keychain storage
│   ├── KeychainService.swift     # DependencyKey — iOS Keychain wrapper (kSecAttrAccessibleWhenUnlockedThisDeviceOnly)
│   ├── KeyValueStorageService.swift # DependencyKey — UserDefaults wrapper
│   └── EvmProviderClient.swift   # DependencyKey — EthereumProvider factory
└── Shared/
    ├── DesignSystem.swift        # Design tokens (Color.x*, Font.x*, XSpacing, XRadius, Animation.x*)
    ├── ActionButton.swift        # Reusable button component
    ├── AuroraBackground.swift    # Animated gradient background
    ├── FloatingTabBar.swift      # Custom tab bar
    ├── Color+Hex.swift           # Color hex initializer
    └── View+RoundedCorner.swift  # Corner radius modifier

xWalletTests/                     # Test target (Swift Testing framework)
├── AccountTests.swift            # Account reducer tests
├── WalletTests.swift             # Wallet reducer tests
└── SettingsTests.swift           # Settings reducer tests

docs/thinking/                    # Design docs & roadmap (gitignored, not shipped)
```

## Dependencies (SPM via Xcode)

- `swift-composable-architecture` — pointfreeco TCA
- `GRDB.swift` — SQLite persistence for wallet identities
- `wallet-core` — trustwallet (WalletCore v4.4.2)
- `MultiChainKit` / `MultiChainCore` / `EthereumKit` — custom multi-chain SDK (BIP-39, signers, providers)
- `BigInt` — large number handling for Wei values

## Build & Test Commands

```bash
# Build (command line)
xcodebuild -project xWallet.xcodeproj -scheme xWallet -destination 'platform=iOS Simulator,name=iPhone 16' build

# Run tests
xcodebuild -project xWallet.xcodeproj -scheme xWallet -destination 'platform=iOS Simulator,name=iPhone 16' test

# Preferred: open in Xcode, Cmd+B to build, Cmd+U to test
open xWallet.xcodeproj
```

No Makefile, Fastfile, or CI pipeline exists yet.

## Architecture Invariants

- Every feature is a TCA `@Reducer` with `@ObservableState` State, Action enum, and `body: some ReducerOf<Self>`.
- Side effects (network, keychain, DB) go through `@Dependency` clients, never called directly from reducers.
- All dependency clients provide `liveValue`, `testValue` (and optionally `previewValue`).
- Tests use `TestStore` from ComposableArchitecture with dependency overrides.
- Private keys and mnemonics are stored ONLY in iOS Keychain (`kSecAttrAccessibleWhenUnlockedThisDeviceOnly`). Wallet metadata lives in SQLite.
- `WalletStorage` is an `actor` — all DB/Keychain access is concurrency-safe.

## Security Constraints

- NEVER log, print, or persist mnemonics/private keys outside Keychain.
- NEVER weaken Keychain accessibility level (must remain `kSecAttrAccessibleWhenUnlockedThisDeviceOnly`).
- NEVER commit `.env` files, API keys, or real private keys.
- Test fixtures use well-known Anvil/Hardhat test keys only (e.g., `0xac0974bec...`).

## Design System Constraints

- All views must use design tokens from `Shared/DesignSystem.swift` — no hardcoded colors, fonts, spacing, or radii.
- Token prefixes: `Color.x*`, `Font.x*`, `XSpacing.*`, `XRadius.*`, `Animation.x*`.
- Card styles: `.xCard()` (frosted glass) or `.xSolidCard()` (solid dark).

## Change Policy

- Minimal diffs only — do not refactor, rename, or "improve" code outside the scope of the task.
- Do not add comments, docstrings, or type annotations to unchanged code.
- Do not create new files unless the task requires it.
- All new reducers must have corresponding tests in `xWalletTests/`.
- Run `xcodebuild test` (or Cmd+U) before considering work complete.
- `Send.swift` is WIP — its reducer body is intentionally incomplete.
- `#if DEBUG _printChanges()` in AppFeature is intentional for development.

---
> Source: [iossocket/xWallet](https://github.com/iossocket/xWallet) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-01 -->
