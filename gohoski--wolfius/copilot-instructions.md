## wolfius

> Wolfius is an Android app (minSdk 3, targetSdk 25) that provides system-wide TLS 1.3 access by performing TLS 1.0 MITM proxying. It uses wolfSSL for TLS 1.2/1.3 and lwIP for VPN tunneling.

Wolfius is an Android app (minSdk 3, targetSdk 25) that provides system-wide TLS 1.3 access by performing TLS 1.0 MITM proxying. It uses wolfSSL for TLS 1.2/1.3 and lwIP for VPN tunneling.

## Build Environment

- Android Studio 2.3.2, SDK 25, build tools 25.0.0
- NDK r10e (uses `ndk-build`, not CMake)
- Gradle 3.3 with AGP 2.3.2
- **Do not run build commands** - user builds manually

## Architecture

### Connection Methods (mutually exclusive)

1. **Root iptables** - Redirects ports 443/80/DNS via NAT to local proxy. Requires netfilter support.
2. **HTTP proxy** - Manual proxy config. Works everywhere but doesn't intercept all apps.
3. **VpnService** (API 14+) - Uses lwIP to intercept traffic via TUN interface.
4. **PPTP VPN** - Root required. Standalone native binary for ROMs without netfilter.

### Native Libraries (built via ndk-build)

- `libwolfssl.so` - TLS operations (SNI parsing, cert generation, handshake)
- `liblwip.so` - lwIP TCP/IP stack for VPN tunneling
- `libpptp_vpn.so` - PPTP VPN client (renamed from `pptp_vpn` executable in post-build hook)

### Key Java Classes

- `ProxyService` - Main service, starts TlsProxy on port 7998, manages iptables/VPN
- `TlsProxy` - Accepts connections, MITMs TLS via wolfSSL, bridges upstream
- `WolfClient` - JNI bridge to wolfSSL native functions
- `WolfSSLSocket` - SSLSocket implementation wrapping wolfSSL native pointers
- `MitmKeyStoreManager` - Generates/installs MITM CA, creates per-host server contexts
- `LwipBridge` - JNI bridge for lwIP VPN mode
- `DnsForwarder` - UDP DNS forwarding (port 5353) with IP-to-hostname reverse mapping

### Native Bridge Pattern

Java ↔ JNI (`wolfssl_bridge.c`, `lwip_bridge.c`) ↔ wolfSSL/lwIP C libraries. Native pointers (SSL*, CTX*) are passed as `long` between Java and JNI.

## Key Configuration Files

- `app/src/main/jni/Android.mk` - ndk-build targets: wolfssl, lwip_static, lwip, pptp_vpn
- `app/src/main/jni/Application.mk` - Default ABI: armeabi, platform: android-3
- `app/src/main/jni/user_settings.h` - wolfSSL compile flags (TLS 1.0-1.3, cert gen, SNI)
- `app/src/main/jni/lwipopts.h` - lwIP tuning (NO_SYS=1, TCP window sizes, checksum offload)
- `app/build.gradle` - ABI splits, ProGuard, post-build pptp_vpn renaming

## Port Allocation

- **7998**: Local TLS proxy listen port (all methods)
- **7999**: UDP resolver for PPTP mode (destination lookup)
- **5353**: DNS forwarding (iptables mode)
- **15000-25000**: Outbound source port range for upstream connections

## wolfSSL Notes

- Custom random seed via `/dev/urandom` (avoids libc filesystem deps)
- `user_settings.h` enables: TLS 1.0/1.2/1.3, cert gen, SNI, ECC, AES-GCM, SHA-384/512
- Per-host server contexts cached in `MitmKeyStoreManager` to avoid repeated cert generation
- Certificate generation is mutex-protected (non-reentrant RSA buffers in C)

---
> Source: [gohoski/Wolfius](https://github.com/gohoski/Wolfius) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-25 -->
