## unite-2-man

> Recovery and stabilization rule for CWSP endpoint, AirPad routing, clipboard sync, TLS/HTTPS, direct/proxy/reverse connectivity, Android/PWA integration, and diagnostics.


# CWSP / AirPad / Clipboard Recovery Rule

## Primary objective
When working in this repository, prioritize recovery and stabilization of the following system capabilities:

- CWSP endpoint correctness
- module/import/runtime integrity
- HTTPS/TLS support
- direct / proxied / reverse connectivity
- AirPad routing and settings persistence
- clipboard synchronization
- cursor/input movement routing
- Android and PWA compatibility
- PM2 / ADB / logcat diagnostics and operational visibility

---

## Mandatory execution model
Always work in this order:

1. audit current implementation
2. identify actual architecture and data flow
3. make targeted corrections
4. run compatibility tests
5. inspect logs and diagnostics
6. iterate further if needed
7. document final state

Do not perform blind edits without understanding root cause and impact.

---

## Mandatory pre-change checks
Before editing related code, inspect all relevant parts of the system:

- `package.json`
- `main`, `module`, `type`, `exports`, `bin`
- start/build/dev scripts
- ESM/CJS compatibility
- import/export paths
- runtime entrypoints
- server/client bootstrap flow
- websocket/network transport layers
- HTTPS/TLS configuration
- reverse proxy behavior
- routing by destination client ID
- AirPad settings persistence
- clipboard event flow
- cursor/input event flow

---

## Architectural constraints
Never conflate the following concepts:

- Endpoint URL
- direct target URL
- AirPad target URL
- destination client ID

The system must support both of these models:

### 1. Direct connect
Example:
- `https://192.168.0.110:8443`

### 2. Endpoint-routed connect
Example:
- `https://192.168.0.200:8443/` with routing toward `L-192.168.0.110`

If an Endpoint URL is configured, it may act as a routing/proxy endpoint rather than the final AirPad device URL.

Where applicable, prefer routing by destination client ID rather than brittle raw-address-only logic.

---

## Known node identities
Preserve and respect the following logical identities:

### `L-192.168.0.110`
Primary AirPad endpoint / internal target endpoint.

### `L-192.168.0.200`
Proxy / endpoint / reverse-proxy node.

### `L-192.168.0.196`
VDS / client / application / PWA side.

Required association:
- `clientId = L-192.168.0.196`
- `token = n3v3rm1nd`

### `L-192.168.0.208`
Additional Android client that may participate in synchronization scenarios.

---

## Security and configuration discipline
Do not permanently hardcode secrets into source unless absolutely necessary.

If values such as the following are required:
- `clientId = L-192.168.0.196`
- `token = n3v3rm1nd`

prefer:
- environment variables
- config files
- documented bootstrap defaults
- secret/config loading logic

If a fallback value remains in code, make it explicit and well documented.

---

## Mandatory functional scenarios

### CWSP / transport
Any changes must preserve or restore:
- endpoint startup without module/import/runtime failure
- direct mode
- proxied mode
- reverse mode
- compatibility across:
  - `http`
  - `https`
  - `ws`
  - `wss`

### TLS / HTTPS
Endpoint behavior on `:8443` must be correct and testable:
- certificate loading
- certificate chain behavior
- browser/PWA/Android compatibility
- WebSocket upgrade over TLS

### AirPad
AirPad configuration must:
- persist correctly
- remain internally consistent
- support direct mode
- support routed mode via endpoint
- avoid semantic confusion between endpoint and target device

### Clipboard
Clipboard sync between `L-192.168.0.196` and `L-192.168.0.110` is a required scenario.

Keep the design extendable for:
- Android
- PWA
- `L-192.168.0.208`

### Cursor movement / input
Cursor movement and related input events originating from `L-192.168.0.196` must reach `L-192.168.0.110`.

This must be validated across:
- direct path
- proxied path
- reverse path where applicable

---

## Logging and diagnostics requirements
Always consider operational visibility:

- PM2 logs
- PM2 process configuration
- PM2 cleanup/flush when logs are oversized
- ADB diagnostics
- Android logcat
- health checks
- smoke tests

If you change services, startup flow, or connectivity behavior, also add or improve diagnostics commands and document them.

---

## Files and artifacts you may need to create or update
When necessary, create or fix:

- `package.json`
- runtime scripts
- PM2 ecosystem config
- TLS/HTTPS configs
- proxy configs
- AirPad configuration persistence
- integration / compatibility tests
- diagnostics scripts
- docs
- migration notes
- `.env.example`

---

## Quality bar
Do not treat the work as complete unless the following are confirmed where applicable:

- CWSP endpoint is repaired
- module/import issues are repaired
- HTTPS/TLS is restored
- direct connection works
- routed/proxy connection works
- clipboard sync works
- cursor movement works
- AirPad settings persistence works
- Android/PWA integration is restored or explicitly prepared/documented
- PM2/logging/diagnostics are operational and documented

---

## Required reporting style
After each significant iteration, report:

1. what was found
2. what is being changed
3. why it is being changed
4. how it will be validated
5. what risks remain

In the final report, always include:

- changed files
- changed configs
- added tests/scripts
- compatibility matrix
- validated scenarios
- unresolved scenarios or risks

---

## Android debugging

- ADB usage (`adb` and/or `adb shell`)
- Connect to `192.168.0.196:5555` (defaultly)
- Reading logcat

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/u2re-space) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-16 -->
