## garf

> - When resolving a linter error, always execute the correspondign unit tests before proceeding to the next linter fix.

# Go

- When resolving a linter error, always execute the correspondign unit tests before proceeding to the next linter fix.
- NEVER change manually `go.mod` and `go.sum`, unless explicitly authorized or instructed.
- AVOID using the local `go` installation on the host machine. PREFER to use `rules_go` Go SDK. Example:

Execute: 

```sh
bazel run @rules_go//go -- mod tidy -v
```

Instead of:

```sh
go mod tidy -v
```

- ALWAYS consider refactoring a function.
- AVOID increasing the scope of a function, prefer to create another one instead.
- Write idiomatic Go. Follow Go proverbs.
- ALWAYS write unit tests and make code testable.

## Libraries

- Use the `make` `go-get` target to add third-party dependencies. Example `DEP=github.com/stretchr/testify make go-get`. After run `bazel mod tidy`.
- Use `github.com/jfrog/jfrog-client-go` to interact with JFrog Artifactory REST APIs.
- Use `github.com/google/go-github` to interact with GitHub APIs, if needed.
- Use `github.com/stretchr/testify` for testing. Prefer to use the `require` package.
- Use `github.com/spf13/viper` if you want to enhance CLI configuration.

## Validation

- Once you are done with code changes, run `make test` and confirm all unit tests are passing.
- Once all unit tests are passing, execute `make lint` and fix the findings before finishing it.
- Repeat this process until both pass.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/albertocavalcante) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
