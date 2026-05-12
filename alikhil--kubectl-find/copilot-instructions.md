## kubectl-find

> - Run `pre-commit` before committing

# kubectl fd

## Commit instructions

- Run `pre-commit` before committing
- Commit messages must follow [Conventional Commits v1.0.0](https://www.conventionalcommits.org/en/v1.0.0/)

## Testing instructions

- Run unit tests with `go test -v ./...`

- For integration test use `kind` or another k8s in docker solution, populate needed objects and test.

## PR instructions

- Title format must be the same as commit message format
- Always run `golangci-lint run` and `go test -v ./...` before committing to PR.

---
> Source: [alikhil/kubectl-find](https://github.com/alikhil/kubectl-find) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-06 -->
