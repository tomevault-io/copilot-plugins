## cast

> - When you start a session read mise.toml files to know which apps, runtimes and tools

# Agents

## ✅ Always DO

- When you start a session read mise.toml files to know which apps, runtimes and tools
  should be executed with mise.
- Write tests for any code changes if they do not exist.
- Run tests before completing the task.
- Fix any broken tests
- Keep the project structure section up to date. Focus only on directories
  and key files such as go.mod, castfile, etc. 
- If you write scripts that need to stay around, save them to eng/scripts.
- If you write scripts that are temporarily, save them to eng/tmp.
- For ci/cd, store artifacts in the .artifacts folder.  
- Store product requirement documents in the docs/prd folder. 
- Use the gh cli to interact with github and github.com/frostyeti projects.
- Create seperate files for integration tests and use the `// +build integration` directive
  and use the `go test -tags=integration ./...` command to run them
- Use testcontainers for go if they are needed for external dependencies.
- Store e2e tests in the `test/e2e` folder and write the tests so that they build cast
  generate files, directories, etc as needed and then run tests, and remove files/and folders
  when done.
- always avoid dependencies that require CGO.
- run govulncheck and fix any vulnerabilities before completing the task.
- use go mod tidy to keep the go.mod and go.sum files clean and up to date.
- use go fmt to keep the code formatted and consistent.
- use go lint to keep the code clean and idiomatic.
- use go doc comments to document any exported functions, types, etc.
- use go generate to generate any code that can be generated, such as mocks, etc.
- use go releaser to automate releases and versioning.
- When editing JSON schemas, keep them user friendly: preserve useful `description` fields, enums, examples, `anyOf`/`oneOf` shapes, and pattern constraints; prefer validating multiple supported YAML forms instead of collapsing them into `type: object`.
- When a YAML key accepts more than one shape in code, model that explicitly in the schema and docs, and call out any compatibility or experimental caveats instead of removing the validation.
- Document naming rules and common values in schema-friendly docs so users can discover valid `uses`, ids, aliases, and optional markers without reading the code.
- When making new alpha/prerelease releases, DO NOT bump the minor version (e.g. v0.2.0-alpha.0). Instead, increment the alpha version number (e.g. v0.1.0-alpha.1, v0.1.0-alpha.2, etc).

## Project Structure


```text
.
├── AGENTS.md
├── castfile            # Example project definition
├── cmd/                # CLI commands
├── docs/               # Documentation
├── go.mod              # Go module definition
├── internal/           # Internal packages
│   ├── errors/
│   ├── eval/
│   ├── id/
│   ├── logx/
│   ├── modules/
│   ├── paths/
│   ├── projects/
│   ├── runstatus/
│   ├── scriptx/
│   ├── types/
│   └── web/            # Web server daemon
├── LICENSE
├── main.go             # Main application entry point
├── mise.toml           # Tool versions configuration
├── schemas/            # JSON schemas
├── test/
│   └── e2e/            # End-to-end testing
└── ui/                 # Svelte embedded web UI
```

---
> Source: [frostyeti/cast](https://github.com/frostyeti/cast) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-01 -->
