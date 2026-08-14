## bwenv

> `bwenv` is a Go CLI wrapper around the official Bitwarden Secrets Manager CLI (`bws`). It lets one Bitwarden project represent an environment containing many application namespaces.

# bwenv — agent guide

## Product

`bwenv` is a Go CLI wrapper around the official Bitwarden Secrets Manager CLI (`bws`). It lets one Bitwarden project represent an environment containing many application namespaces.

```text
<app>__KEY   → immich__DATABASE_URL
shared__KEY  → shared__TZ
```

The CLI strips prefixes for output and process environments. With `--include-shared`, shared values load first and app values override them.

## Command surface

```text
bwenv create <app> <key> <value>
bwenv import <app> <file|->
bwenv list <app>
bwenv get <app> <key>
bwenv edit <app> <key>
bwenv delete <app> <key>...
bwenv export <app>
bwenv run <app> -- <command>
bwenv completion <shell>
bwenv version
```

Do not add an `env` command group: the product name already supplies that context. Do not wrap `bws project` or `bws config`.

Official global options are `--output`, `--color`, `--access-token`, `--config-file`, `--profile`, and `--server-url`. `--project-id` and `--verbose` are bwenv options. `--include-shared` appears only on read/export/run commands; `--dry-run` appears only on mutations.

## Architecture

- `main.go` translates `cmd.Execute()` into a process exit code.
- `cmd/` keeps each flat command in its own `<command>.go` file and uses Cobra's built-in completion command. Shared construction helpers live in `environment.go`; avoid package-global command state.
- `internal/bws/` is the sole Bitwarden I/O boundary and shells out to `bws` with `exec.CommandContext`.
- `internal/environment/` owns validation, prefix mapping, resolution, duplicates, sorting, and shared precedence.
- `internal/output/` renders normalized records after filtering; it does not contact Bitwarden.

Internal Bitwarden calls force JSON and disable color, then bwenv applies its environment transformation. Do not reimplement authentication or call Secrets Manager HTTP APIs directly.

## Security invariants

- Never log or return secret values, notes, or access tokens in bwenv-generated errors.
- Mask access tokens, values, and notes in `--verbose` subprocess logs.
- Remove `BWS_ACCESS_TOKEN` from `run` children.
- Reject null bytes and ambiguous duplicate full keys.
- Preserve empty values and values beginning with `-`.
- Resolve every requested delete before mutation.
- Values passed to official create/edit commands are visible in local process lists; document rather than conceal this limitation.

## Development

Go version and dependencies are declared in `go.mod`.

Use feature branches off `main` and open pull requests. Never push directly to `main`.

```bash
git checkout -b fix/my-change
# make changes, commit
git push origin fix/my-change
# open PR at the URL shown
```

```bash
make verify       # formatting, vet, unit tests, shell syntax, doc links
make test-race    # race detector
make build        # current platform with version metadata
make build-all    # supported release targets
make lint         # golangci-lint when installed
```

Tests must not require a real `bws` binary, token, or network. Command tests inject a fake client. Add cases for success, validation, remote failure, redaction, and exit status. Run Windows cross-compilation for process changes.

## Delivery

- `install.sh` downloads raw release binaries and requires SHA-256 verification.
- `Formula/bwenv.rb` is a valid HEAD formula until stable formula publishing is automated.
- CI tests Linux, macOS, and Windows. Release tags `v*` build five targets, embed metadata, smoke-test, checksum, and publish.
- Never commit generated binaries, `dist/`, access tokens, or exported dotenv files.

## Documentation

`docs/` is authoritative for humans and coding agents. `docs/README.md` indexes every current document. Update the CLI reference and architecture with behavior changes, add ADRs for durable product decisions, remove stale audits, and run the link checker. Keep the root README concise and user-focused.

---
> Source: [saiteja-madha/bwenv](https://github.com/saiteja-madha/bwenv) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-13 -->
