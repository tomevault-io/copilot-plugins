## openapi-generation

> Before creating or publishing any public artifact — PR titles/bodies, comments, review replies, commit messages, changesets, branch/file names, code comments and string literals, fixtures, docs — follow `.claude/skills/public-repo-communication/SKILL.md`: describe the change so it stands alone; never expose customer-derived identifiers, private paths/trackers, or workflow provenance.

Before creating or publishing any public artifact — PR titles/bodies, comments, review replies, commit messages, changesets, branch/file names, code comments and string literals, fixtures, docs — follow `.claude/skills/public-repo-communication/SKILL.md`: describe the change so it stands alone; never expose customer-derived identifiers, private paths/trackers, or workflow provenance.

## Definitions

### Targets

A `TARGET` is one of:
- cli
- csharp
- go
- javav2
- mcp-typescript
- php
- postman
- pythonv2
- ruby
- terraform
- typescriptv2
- unity

Their associated *templates* directory is located at `templates/templates/<TARGET>`

### Variants

A `VARIANT` is one of:
- basic-http
- client-credentials
- client-credentials-basic
- custom-http
- no-servers
- no-zod
- oauth2-password
- primary
- quaternary
- relative-servers
- review
- secondary
- security-options
- tertiary

Their associated *configuration* is located at: `tests/config/<VARIANT>/<TARGET>/.speakeasy/gen.yaml`

Note: in the following command `TARGET=review make test-typescriptv2`, the term `TARGET` is misleading and should have been called `VARIANT` instead (see Makefile#L35)

### Specs

- The main test specification is `tests/specs/uber.yaml` and directly drives the `primary` variant.
- Additional test fragments may be applied as configured in `tests/specs/fragments/`.
- Variants apply *overlay* files as defined in `tests/overlays`.


## How to debug an issue:

Workout what the issue is in the generator source code and fix it.

You can see what the issue is and verify you've fixed it by running

go run cmd/generate/main.go -s {{SOME_TMP_DIR}}/openapi.yaml -o {{SOME_TMP_DIR}} -l {{TARGET}} --license agpl-3.0-only

Note: first run will automatically create a gen.yaml under .speakeasy with default values.

## License election for local generation

Every generation needs an explicit license election. Run `./zero` once per
checkout: `./zero --license agpl-3.0-only` (OSS / agents) or `./zero` after
`speakeasy auth login` (commercial token). It writes a gitignored `.env` that
`make`, `mise run`, and `scripts/*.sh` load. When running `go run
cmd/generate/main.go` or `cmd/regen` directly, pass `--license agpl-3.0-only`
or have `SPEAKEASY_LICENSE_TOKEN` / `SPEAKEASY_GENERATED_LICENSE` in the
environment (a mise-activated shell loads `.env`).

Output written inside this repository (`zSDKs/`, `testSDKs/`) never carries
license headers, `LICENSE`, or `NOTICE` regardless of the election — the
repository LICENSE covers it — so regenerating fixtures produces the same
bytes for every contributor. Never commit `.env` or a license token.

### Regenerating an already-bootstrapped SDK

If the SDK directory already has a `.speakeasy/gen.yaml` (and optionally a
`workflow.yaml`), use `cmd/regen` which infers language and schema automatically:

```
go run cmd/regen/main.go {{SDK_DIR}}
```

- **Language** is inferred from the gen.yaml language key
- **Schema** is inferred from workflow.yaml, then by scanning for `openapi*`
  files in the SDK dir and up to 3 parent directories
- Override with `-s spec.yaml` or `-l go` if needed
- Patch gen.yaml before generation: `--set key=value` (e.g. `--set go.version=2.0.0`)

**Note:** `go run -C` changes the process cwd, so `.` resolves to the module
dir, not your shell cwd. Use absolute paths, or build a binary first:
`go build -o /tmp/regen ./cmd/regen && cd {{SDK_DIR}} && /tmp/regen .`


## You've fixed the issue, now run the tests

### SDK Testing

Run the tests for the SDKs:
  - `TARGET=review make test-{{TARGET}}` (outputs to `zSDKs/sdk-{{TARGET}}`)
  - `TARGET=primary make test-{{TARGET}}` (outputs to `testSDKs/sdk-{{TARGET}}-primary`)

You can see how tests are configured under `tests/*` eg the spec for the primary,secondary etc tests is in `tests/specs/uber.yaml` and for review tests it's in `tests/specs/review.yaml`. The gen.yaml for {{TARGET}} primary is
`tests/config/primary/{{TARGET}}/.speakeasy/gen.yaml`.

### Test Fragments

Isolated test cases can be added as OpenAPI Specification document fragments under `tests/specs/fragments/primary/` (or `tests/specs/fragments/uber/` for shared fragments). Fragments get merged into the base spec during build via `scripts/build-openapi-document.sh`.

- Use **kebab-case** filenames (e.g., `pagination-nullable-limit.yaml`).
- Include a `description` on the operation or schema under test explaining what the fragment is intended to check and why. Descriptions should be language-agnostic, focusing on the OpenAPI construct being tested rather than language-specific generated output.
- Ensure parameter and property names align with their testing purpose or defined characteristics (e.g., a schema with nullable typing should be named `NullableObject`).
- Include the standard security block matching other fragments.

### API Test Service

The `services/speakeasy-api-test-service/` directory contains a Go HTTP server used for runtime testing of generated SDKs. It provides customized request/response behavior for scenarios that cannot be tested via httpbin or simple mock server reflection, such as pagination, retries, event streams, polling, auth flows, and error handling.

- **Handlers** live under `services/speakeasy-api-test-service/internal/` organized by feature (e.g., `pagination/`, `retries/`, `eventstreams/`).
- **Routes** are registered in `services/speakeasy-api-test-service/cmd/server/main.go`.
- **Rebuild** with `make build-api-test-service`. If the service is already running from a previous test run, kill it first (`pkill -f 'bin/api-test-service'`) so `make test-{{TARGET}}` restarts it with the new binary.
- When adding a new fragment that needs runtime testing, add a `servers` block with `url: http://localhost:35456` to the operation so the generated SDK calls the test service instead of the default SDK server URL.
- Additional runtime tests live in `templates/templates/{{TARGET}}/tests/primary/` using the `_additional` naming convention (e.g., `pagination_additional_test.go`). Some targets, such as PHP, use uppercase `Tests` directory (e.g., `templates/templates/php/Tests/primary/`).

### Terraform Testing

Terraform testing is primarily done using the generated review provider code in `zSDKs/terraform-provider-testing`. The review provider OpenAPI source is in `tests/specs/review-terraform.yaml` and its generation configuration is in `tests/config/review/terraform/.speakeasy/gen.yaml`.

Use `./scripts/build-review-terraform.sh` to rebuild the review provider and run Terraform acceptance testing. Check for Git differences to confirm the output matches expectations before investigating individual test failures.

To run individual Terraform acceptance tests, use `TF_ACC=1 go test -count=1 -run`.

## Writing files from JS templates

- **`templateFile()`** should be preferred for code output (`.tf`, `.go`, `.ts`, etc.). It renders a `.stmpl` template and routes through the formatting pipeline (`format.Format()`), which applies target-specific formatters (e.g., HCL formatter for `.tf` files, `gofmt` for `.go` files).
- **`writeFile()`** should be reserved for 1:1 content preservation where formatting is intentionally not desired. It calls `cfg.WriteFileFunc` directly, bypassing the formatting pipeline.

## Editing shell scripts under scripts/

Scripts in `scripts/` must run on both macOS (BSD userland) and Linux (GNU
userland). The most common pitfall is `grep`:

- **Use `grep -E`** (POSIX extended regex), never `grep -P` (PCRE). The `-P`
  flag is GNU-only and silently fails on macOS's BSD grep, producing no output.
- **To extract a capture group**, pipe through `cut` or `sed` instead of using
  `\K` (PCRE lookbehind reset), e.g. `grep -Eo 'KEY=[0-9]+' | cut -d= -f2`.
- **Avoid GNU-only `sed` extensions** (`-r` is `-E` on macOS; prefer `awk` for
  anything non-trivial).
- **Avoid `date -d`** (GNU-only); use `python3` if date arithmetic is needed.

## You've fixed the issue and the tests pass. Before submitting the PR for review:


- Follow [CONTRIBUTING.md](CONTRIBUTING.md) guidelines
- Run `make lint`
- If this change can have a meaningful effect on generated SDKs you should run `go run cmd/changelog/main.go`
- If any changes added or removed files in `templates/`, run `go generate ./...` to ensure the file permissions checks are updated.
- If any changes affected `templates/templates/` files, run `npm run format` to apply prettier formatting, then run `make check-template-{{TARGET}}` to ensure the TypeScript compiler checks still pass.

---
> Source: [speakeasy-api/openapi-generation](https://github.com/speakeasy-api/openapi-generation) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-18 -->
