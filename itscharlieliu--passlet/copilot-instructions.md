## passlet

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```sh
# Run PassletCore unit tests (no Xcode needed)
swift test

# Run a single test
swift test --filter KDBXRoundTripTests/testCreateAndRoundTripV4

# Build PassletCore only
swift build

# Generate the full Xcode project (requires xcodegen: brew install xcodegen)
xcodegen generate
open Passlet.xcodeproj
```

The `Package.swift` only covers `PassletCore`. The iOS app and AutoFill extension targets are defined in `project.yml` and require `xcodegen generate` to produce `Passlet.xcodeproj`.

## Architecture

Three targets share the same repo:

- **`PassletCore`** – Pure Swift framework (no UIKit). Shared by the app and the AutoFill extension. Compiled from `PassletCore/Sources/PassletCore/` and `Shared/`.
- **`Passlet`** – iOS app (`Passlet/Sources/Passlet/`). Imports `PassletCore`.
- **`PassletAutoFill`** – `ASCredentialProviderExtension` (`PassletAutoFill/Sources/PassletAutoFill/`). Imports `PassletCore` but does **not** run Argon2 (extension memory cap ~120 MB; 64 MiB Argon2 is borderline).

### KDBX read/write pipeline

`KDBXReader.read(data:compositeKey:)` dispatches on major version:

**KDBX4 path:**
1. Parse outer header → `KDBXHeader` (TLV fields, VariantDictionary KDF params)
2. Verify SHA-256 header hash + HMAC-SHA-256 header MAC
3. `HMACBlockStream.readAll` → each block has its own HMAC key derived as `SHA-512(blockIndex_LE64 + hmacKey)`
4. `BlockCipher.decrypt` (AES-256-CBC or ChaCha20) using `keys.encryptionKey` and `encryptionIV`
5. Parse **inner header** from raw decrypted bytes (NOT decompressed — only the XML body is compressed)
6. `gunzipped()` on the remaining bytes → XML
7. `KDBXXMLParser` (SAX) walks the XML; protected `<Value>` fields are XOR-decrypted via `InnerStreamCipher`

**KDBX3.1 path:** similar but uses `HashedBlockStream` (SHA-256 per block), `StreamStartBytes` to verify the key, and the inner stream cipher parameters come from the outer header.

`KDBXWriter` is the reverse of the above.

### Key derivation (`Crypto/KeyDeriver.swift`)

```
compositeKey = SHA256(SHA256(password))          # per credential, then SHA256(concat)
transformedKey = Argon2d/id(compositeKey, ...)   # or AES-KDF for KDBX3.1
encryptionKey  = SHA256(masterSeed + transformedKey)
hmacKey        = SHA512(masterSeed + transformedKey + 0x01)
blockHMACKey[i] = SHA512(uint64_LE(i) + hmacKey)  # header uses i=0xFFFF...FFFF
```

### AutoFill credential flow

The extension never touches `.kdbx`. Instead, after each successful unlock/save:
1. `DatabaseService` builds `[CredentialIndexEntry]`, encrypts with AES-256-GCM, writes to the shared App Group container (`group.com.example.passlet/credential_index.enc`)
2. The AES key lives in the shared keychain with `kSecAttrAccessibleAfterFirstUnlock`
3. `ASCredentialIdentityStore.shared.saveCredentialIdentities(...)` populates the QuickType bar
4. The extension reads the index and calls `extensionContext.completeRequest(withSelectedCredential:)`

### Merge (`Merge/DatabaseMerger.swift`)

On save, `DatabaseService` re-reads the file and calls `DatabaseMerger.merge(local:remote:)` before writing:
- Same UUID in both: last `LastModificationTime` wins; loser's state pushed to `entry.history`
- Only in remote: added (unless local `DeletedObjects` has a newer deletion time)
- Only in local: kept (unless remote `DeletedObjects` has a newer deletion time)
- `DeletedObjects`: union, keeping later deletion time per UUID

### Notable implementation details

- **Salsa20** is not in CryptoSwift — implemented in `Crypto/Salsa20.swift` using explicit local variables to avoid Swift exclusivity violations in the core loop.
- **GZip** uses raw `zlib` (available on Apple platforms without import). Input must be copied into a stable `UnsafeMutableBufferPointer` before setting `stream.next_in`, because `withUnsafeBytes` closures do not keep pointers valid after they return.
- `CryptoSwift`'s `Data.sha256()` returns `Data` (not `[UInt8]`). Use it directly; don't redefine it.
- `Argon2Swift.hashPasswordBytes(password:salt:iterations:memory:parallelism:length:type:version:)` takes `Data` and `Salt(bytes: Data)` — not `[UInt8]`. `memory` is in KiB.

## Bundle ID / App Group configuration

Three places must be updated consistently before running on a real device:
1. `project.yml` → `bundleIdPrefix`
2. `Passlet/Passlet.entitlements` and `PassletAutoFill/PassletAutoFill.entitlements` → `com.apple.security.application-groups`
3. `Shared/CredentialIndex/CredentialIndexStore.swift` → `appGroupID`

Security-scoped bookmarks (for re-opening files without the picker) are stored in the main app's own `UserDefaults` — **not** the App Group defaults, because the extension cannot resolve them.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/itscharlieliu) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
