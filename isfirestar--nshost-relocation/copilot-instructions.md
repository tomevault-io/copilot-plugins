## nshost-relocation

> `nshost` (`libnshost.so`, version 9.8.6) is a C, epoll-driven asynchronous network I/O

# nshost

`nshost` (`libnshost.so`, version 9.8.6) is a C, epoll-driven asynchronous network I/O
library for TCP/UDP/ARP. Public C API lives in `libnsp/icom/nis.h` and `nisdef.h`.
The POSIX implementation is under `posix/`; shared runtime code is under `libnsp/com/`.
`demo/` contains the only runnable app: `nshost.echo`, a TCP echo server/client.
`win32/` is the Windows (Visual Studio) build and is not used on Linux.

## Cursor Cloud specific instructions

Toolchain (`build-essential` = gcc/make, `libcrypt-dev`) is already present in the base
image and is refreshed by the startup update script. `cc` resolves to clang on this image,
which builds the C89 library and gnu99 demo fine (only warnings, no errors). There are no
automated tests and no linter config; "lint" is effectively compiling with `-Wall`.

### Build the shared library

```
cd posix && make            # produces posix/libnshost.so + libnshost.so.9.8.6
```

### Install so the demo can link against it (non-obvious)

The demo Makefile links `/usr/local/lib64/nshost.so`, but `posix/make install` produces
`libnshost.so` (with the `lib` prefix) in `/usr/local/lib64`. You must also create a
`nshost.so` name there. This is a one-time environment step (needs sudo):

```
sudo mkdir -p /usr/local/lib64
cd posix && sudo make install
sudo ln -sf /usr/local/lib64/libnshost.so.9.8.6 /usr/local/lib64/nshost.so
echo /usr/local/lib64 | sudo tee /etc/ld.so.conf.d/nshost.conf
sudo ldconfig
```

### Build the demo

Plain `make` in `demo/` FAILS with undefined `tcp_*` symbols — the default target only
compiles `libnsp/com/*.c`, not the `posix/` network stack. Always build with `nshost=yes`
to link the installed shared library:

```
cd demo && make nshost=yes   # produces demo/nshost.echo
```

### Run + demonstrate (echo over TCP)

Default port is `10256`; the server binds `0.0.0.0`.

```
# terminal 1 (server)
cd demo && ./nshost.echo -s -p 10256

# terminal 2 (client)  — see gotchas below
cd demo && ./nshost.echo -c127.0.0.1 -p 10256
```

Gotchas when driving the client:
- `-c`/`-s` take an OPTIONAL getopt argument, so the host must be ATTACHED:
  use `-c127.0.0.1`, NOT `-c 127.0.0.1` (a space leaves host at `0.0.0.0` and the client
  prints usage and exits). `-p <port>` (required arg) accepts a space.
- `tcp_connect2` is asynchronous. Let the connection establish before writing, otherwise
  the process can exit before the async write flushes. When feeding stdin non-interactively,
  add a short delay first, e.g.
  `(sleep 1; printf 'hello\n'; sleep 3) | ./nshost.echo -c127.0.0.1 -p 10256`.
- The server prints `[income <ip>:<port>] <data>` on receive. Note the server's echo-back
  branch is unreachable due to a `display()` return-value check in `demo/echo.c`, so it
  logs the received data and closes the link rather than echoing it back.
- Log output to stdout uses raw `write()` (unbuffered); no `fflush` needed.

---
> Source: [isfirestar/nshost-relocation](https://github.com/isfirestar/nshost-relocation) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
