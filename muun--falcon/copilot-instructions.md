## falcon

> Local backend for the mobile apps (Android & iOS). Treat it like backend code.

# Libwallet

Local backend for the mobile apps (Android & iOS). Treat it like backend code.

## Conventions

The conventions for this codebase are documented in the review guidance, which is also what the
automated PR review uses. Read these before writing or reviewing libwallet code:

- **General Go**: `../.github/review/sources/go/review-go-style.md`: general Go style conventions that apply to all Go projects.
- **Libwallet**: `../.github/review/sources/go/review-libwallet-conventions.md`: Libwallet specific conventions.

The canonical upstream is the Notion "Go style-conventions guide" and "Libwallet conventions guide"
pages, cited from those files.

## Build & Test

Run all tests with:
```bash
cd libwallet && go test ./...
```

When modifying tests or logic covered by tests, run the affected tests:
```bash
go test ./path/to/affected/package/...
```

After applying any Go code changes, always run:
```bash
go vet ./...
go fmt ./...
```

---
> Source: [muun/falcon](https://github.com/muun/falcon) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-25 -->
