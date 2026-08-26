## crlf-powered-desync-scanner

> This is a Burp Suite Extension using the Montoya API. The extensions purpose is to detect HTTP header injection vulnerabilities via CRLF injections.

# CLAUDE.md
This is a Burp Suite Extension using the Montoya API. The extensions purpose is to detect HTTP header injection vulnerabilities via CRLF injections.

## Architecture

- **Main Entry Point**: `src/main/kotlin/BurpExtender` - implements `BurpExtension` interface
- **Build System**: Gradle with Kotlin DSL, Java 21 compatibility
- **Dependencies**: Montoya API (latest version) (compile-only), BulkScan (latest version) which can be found at https://github.com/albinowax/bulkscan

## Key Development Commands

```bash
./gradlew build    # Build and test the extension
./gradlew clean    # Clean build artifacts
```

The built JAR file will be in `build/libs/` and can be loaded directly into Burp Suite.

## Extension Loading in Burp

1. Build the JAR using `./gradlew build jar`
2. In Burp: Extensions > Installed > Add > Select the JAR file
3. For quick reloading during development: Ctrl/⌘ + click the Loaded checkbox

## Current State
The extension currently features a number of detection techniques which can be found and disabled/enabled in the ReqMutator file. The main logic for the scan (found in ReqSplit.kt) is fairly complex, repeating probes out of order in order to ensure a consistent change before reporting. Additionally, a followup is executed in order to further improve the reliability of the detection. Several WAF detection rules are also in place in the WAFChecker file, which simply runs through known WAF response strings and checks them against the various responses.

---
> Source: [t0xodile/crlf-powered-desync-scanner](https://github.com/t0xodile/crlf-powered-desync-scanner) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-26 -->
