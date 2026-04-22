## vtest2

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

VTest2 is a standalone HTTP testing program, originally derived from Varnish's varnishtest. It's designed to test HTTP clients, servers, and proxies using a domain-specific scripting language (`.vtc` files). This is the second iteration that maintains better synchronization with Varnish-Cache while being usable independently.

## Build Commands

### Basic Build
```bash
make vtest          # Build without Varnish support
make test           # Build and run all tests
make clean          # Remove build artifacts
```

### Build with Varnish Support
```bash
make varnishtest VARNISH_SRC=/path/to/varnish-cache
```

### Generated Files
The build process generates two files that must exist before compilation:
- `src/teken_state.h` - Generated from `src/sequences` using awk
- `src/vtc_h2_dectbl.h` - Generated from `src/tbl/vhp_huffman.h` using Python

### Syncing with Varnish-Cache
```bash
make update         # Pull updates from Varnish-Cache repository
                    # Set VARNISHSRC env var to use local repo instead of cloning
```

## Code Architecture

### Directory Structure
- **lib/** - Shared utility library code from Varnish-Cache (vsb, vqueue, vtcp, etc.)
- **src/** - Core VTest implementation
  - `vtc_main.c` - Entry point, test execution orchestration, parallel test runner
  - `vtc.c` - Core test execution, macro expansion, command dispatch
  - `vtc_http.c/vtc_http.h` - HTTP/1.1 client/server implementation with session operations abstraction
  - `vtc_http2.c` - HTTP/2 support
  - `vtc_tls.c` - TLS/SSL support (OpenSSL-based)
  - `vtc_client.c` - Client command implementation
  - `vtc_server.c` - Server command implementation
  - `vtc_process.c` - Process spawning and management with terminal emulation
  - `vtc_barrier.c` - Synchronization primitives for tests
  - `vtc_logexp.c` - Log expectation matching (conditional, Varnish-specific)
  - `vtc_varnish.c` - Varnish-specific testing (conditional compilation)
  - `vtc_haproxy.c` - HAProxy-specific testing
  - `builtin_cert.h` - Default self-signed certificate (CN=example.com)
  - `vtc_asn_gentm.c` - OpenSSL compatibility layer for older versions
  - `tbl/` - Table-driven code generation (H2 frames, settings, errors, TLS alerts/protocols)
- **tests/** - Test suite (`.vtc` files)
- **tools/sync/** - Scripts for syncing code with Varnish-Cache

### Command System
Commands are registered via `cmds.h` using X-macros:
- **CMD_GLOBAL**: Available everywhere (barrier, delay, shell, include)
- **CMD_TOP**: Top-level test commands (client, server, process, varnish, haproxy, etc.)

The extensible command system allows loading shared libraries via `-E extension_shlib` to add custom commands at runtime using `add_cmd()`.

### Core Concepts

**Test Execution Model:**
Tests are executed in forked child processes with their own temporary directories. The main process (`vtc_main.c`) orchestrates parallel execution (configurable via `-j`), manages timeouts, and collects results via pipes.

**Macro System:**
VTest has a powerful macro expansion system supporting:
- Static macros defined via `-D name=val`
- Dynamic macros defined in test scripts
- Function macros (e.g., `${date}`, `${string,repeat,N,text}`)
- Automatic macros for servers/clients (sNAME_addr, sNAME_port, sNAME_sock)

**Session Abstraction:**
HTTP clients and servers share common session handling code (`vtc_sess.c`, `sess_process()`) with pluggable connect/disconnect callbacks. This enables code reuse between client, server, and tunneling modes.

**Session Operations Pattern:**
VTest2 uses a session operations abstraction (`struct sess_ops` in `vtc_http.h`) that enables transparent switching between plain TCP and TLS connections. The `struct http` contains operation pointers (`poll`, `read`, `write`, `close`) that dispatch to either plain file descriptor operations (`http_fd_so`) or TLS operations (`tlsconn_so`). This allows all HTTP I/O code to remain unchanged whether using plain or encrypted connections.

**Terminal Emulation:**
Process commands include a full terminal emulator (`teken.c`) supporting VT220 escape sequences for testing interactive programs and screen output.

## Test File Format

Tests are `.vtc` files that must start with `vtest` or `varnishtest` followed by a description:
```vtc
vtest "Test description"

server s1 {
    rxreq
    txresp
} -start

client c1 -connect ${s1_sock} {
    txreq
    rxresp
    expect resp.status == 200
} -run
```

### TLS Test Example
```vtc
vtest "Basic TLS test"

server s1 {
    tls_config {
        cert = "/path/to/cert.pem"
        version = TLSv1.2 TLSv1.3
    }
    tls_handshake
    rxreq
    txresp -status 200
} -start

client c1 {
    tls_config {
        servername = "example.com"
        verify_peer = false
    }
    tls_handshake
    txreq -url "/"
    rxresp
    expect resp.status == 200
    expect tls.version == "TLSv1.3"
} -run
```

## Build Dependencies

### Linux
- libpcre2-dev
- zlib
- libssl-dev (OpenSSL 1.0.x or later, 1.1.0+ recommended)

### macOS
- Libraries typically available via Homebrew
- OpenSSL 3.x (via `brew install openssl@3`)

**Note:** Current Makefile uses hardcoded paths for macOS. For portability, consider using `pkg-config --cflags --libs libssl libcrypto`.

## Platform-Specific Notes

- **Linux**: Uses `dlopen()` by name for extensions. Primary development/testing platform.
- **macOS**: Bound but non-listening sockets timeout instead of refusing, so bad_backend_fd is closed. TLS implementation compiles successfully but may encounter VTest2 runtime issues unrelated to TLS (file descriptor management in `VSUB_closefrom()`). Recommend testing TLS features on Linux.
- **FreeBSD jails**: localhost/127.0.0.1 becomes the jail's IP

## Conditional Compilation

Several features are conditionally compiled:
- `VTEST_WITH_VTC_VARNISH` - Varnish integration (vtc_varnish.c)
- `VTEST_WITH_VTC_LOGEXPECT` - Log expectation matching
- `VTEST_WITH_VTC_VSM` - Varnish Shared Memory support

## TLS/SSL Support

VTest2 includes comprehensive TLS support ported from Varnish Cache Plus. See `TLS-IMPL.md` for detailed implementation documentation.

### Key TLS Features
- TLS 1.0 through TLS 1.3 support (OpenSSL version dependent)
- Certificate loading and validation
- ALPN (Application-Layer Protocol Negotiation)
- Session resumption
- OCSP stapling
- Client certificate verification
- SNI (Server Name Indication)

### TLS Commands
- **tls_config** - Configure TLS settings (certificate, versions, ciphers, ALPN, etc.)
- **tls_handshake** - Perform TLS handshake and switch to encrypted I/O

### TLS Configuration Options

**Common (client & server):**
- `cert = FILENAME` - Load certificate/key bundle (PEM format)
- `version = PROTO_MIN [PROTO_MAX]` - Set TLS version range (SSLv3, TLSv1.0, TLSv1.1, TLSv1.2, TLSv1.3)
- `cipher_list = CIPHER[:CIPHER...]` - Set cipher list for TLS ≤ 1.2
- `ciphersuites = CIPHER[:CIPHER...]` - Set cipher suites for TLS 1.3
- `alpn = PROTO [PROTO...]` - Set ALPN protocol list (e.g., "h2 http/1.1")

**Server-only:**
- `client_vfy = none|optional|required` - Client certificate verification mode
- `client_vfy_ca = FILENAME` - CA bundle for client certificate verification
- `staple = FILENAME` - Provide OCSP staple response

**Client-only:**
- `servername = HOST` - Set SNI hostname
- `verify_peer = true|false` - Enable/disable certificate verification
- `sess_out = filename` - Save TLS session for resumption
- `sess_in = filename` - Resume TLS session from file
- `cert_status = true|false` - Request OCSP staple from server

### TLS Variables for expect Commands

After `tls_handshake`, these variables are available:
- `tls.version` - Negotiated TLS version (e.g., "TLSv1.3")
- `tls.cipher` - Negotiated cipher suite
- `tls.servername` - SNI hostname (client)
- `tls.alpn` - Negotiated ALPN protocol
- `tls.alert` - Latest TLS alert message
- `tls.failed` - "true" if handshake/I/O failed
- `tls.cert[N].subject` - Certificate CN (N=0 is peer cert)
- `tls.cert[N].issuer` - Certificate issuer
- `tls.cert[N].subject_alt_names` - Subject Alternative Names
- `tls.sess_reused` - "true" if session was resumed
- `tls.staple_requested` - "true" if client requested OCSP
- `tls.ocsp_cert_status` - OCSP certificate status
- `tls.ocsp_resp_status` - OCSP response status
- `tls.ocsp_verify` - OCSP signature verification result

### TLS Architecture

The TLS implementation uses a **session operations abstraction** pattern that allows seamless switching between plain TCP and TLS connections:

1. `struct sess_ops` defines function pointers for `poll`, `read`, `write`, `close`
2. `struct http` contains operation pointers that default to plain FD operations
3. After `tls_handshake`, operations switch to TLS implementations
4. All HTTP I/O code remains unchanged - it transparently uses the correct operations

This design cleanly separates TLS concerns from HTTP protocol handling.

## Development Notes

- Code in `lib/` is synced from Varnish-Cache - don't modify directly
- Extensions use `add_cmd(name, function, flags)` to register new commands
- Test names in `tests/` follow pattern `a[0-9]{5}.vtc`
- Use `leave_temp` options (`-l`, `-L`) to preserve test directories for debugging
- The codebase uses Varnish's utility libraries: miniobj for object checking, vsb for string buffers, vqueue for linked lists
- TLS support requires OpenSSL; version detection happens at compile-time via `OPENSSL_VERSION_NUMBER`
- Default self-signed certificate (CN=example.com) is embedded in `builtin_cert.h` for testing without providing certificates

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/varnish) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
