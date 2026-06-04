## terraform-provider-trocco

> This document provides the MOST IMPORTANT information for executing tasks. Before executing a task, you MUST read this document and follow the instructions COMPLETELY. NEVER forget and ignore any of the instructions.

# CLAUDE.md

This document provides the MOST IMPORTANT information for executing tasks. Before executing a task, you MUST read this document and follow the instructions COMPLETELY. NEVER forget and ignore any of the instructions.

## Overview

This repository provides a Terraform provider for [TROCCO](https://trocco.io), a cloud ETL service.

## Dependencies

- Terraform Plugin Framework

## Commands

### Foramtting

```sh
# Format all Go files.
go fmt ./...

# Format a specific Go file.
go fmt [FILE]

# Format a Terraform configuration files.
terraform fmt [FILE]
```

### Testing

```sh
# Run all acceptance tests.
make testacc

# Run specific acceptance tests.
make testacc TESTARGS="-run TestAccSome"

# Run unit tests.
go test -v -cover ./...
```

## Instructions

You MUST use English in files and pull requests.

---

You MUST format Go files using `golangci-lint run --fix`.

You MUST format Terraform configuration files using `terraform fmt`.

---

You MUST follow Conventional Commits in commit messages and PR titles.

---

You MUST use lowercase letters for log messages.

---
> Source: [trocco-io/terraform-provider-trocco](https://github.com/trocco-io/terraform-provider-trocco) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-04 -->
