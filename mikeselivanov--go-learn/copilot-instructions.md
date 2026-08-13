## go-learn

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repository is

A personal Go-learning sandbox (module `example.com/ms-demo-1`) used to work through exercises from the Go Tour and general language features. It is not a product codebase — there is no stable API or long-term architecture to preserve.

`main.go` is a scratchpad: each commit typically rewrites its contents wholesale to explore a new topic (generics, channels/goroutines, the concurrent web crawler exercise, etc.), rather than incrementally extending prior code. Don't assume continuity of `main.go`'s contents across commits, and don't be surprised if a change replaces most of the file — that matches how this repo is used. The `mascot` package is a separate, stable package used to practice basic package/test structure and is not part of the scratchpad churn.

## Commands

- Build: `go build ./...`
- Run the main program: `go run main.go`
- Run all tests: `go test ./...`
- Run a single test: `go test ./mascot -run TestMascot -v`
- Format: `gofmt -l .` (or `gofmt -w .` to fix)
- Vet: `go vet ./...`

## Structure

- `main.go` — single-file scratchpad for whatever exercise is currently being explored; currently implements the Go Tour concurrent web crawler exercise (`Crawl`, `Fetcher`, `fakeFetcher`).
- `mascot/` — small standalone package (`mascot.BestMascot`) with its own test (`mascot_test.go`), used to practice Go package/test conventions.
- `.vscode/launch.json` — VS Code debug configs for running `main.go` directly or via package auto-detection.

---
> Source: [mikeselivanov/go-learn](https://github.com/mikeselivanov/go-learn) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
