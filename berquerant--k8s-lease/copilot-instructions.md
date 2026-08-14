## k8s-lease

> This document is a guide for AI agents and developers to understand and work with this repository.

# AGENTS.md

This document is a guide for AI agents and developers to understand and work with this repository.

---

## Project Overview

**k8s-lease** is a tool that implements **distributed mutual exclusion locking** for shell scripts and command-line tools using Kubernetes [Lease](https://kubernetes.io/docs/concepts/architecture/leases/) resources (`coordination.k8s.io/v1`).

The `klock` command is useful when you want to prevent concurrent execution of the same command across multiple pods or nodes (e.g., preventing duplicate runs of a CronJob).

---

## Repository Structure

- cmd/klock/        : Entry point for the klock command (main.go)
- lease/            : Core logic: lock control using Lease resources
- process/          : External process execution and signal handling
- kconfig/          : kubeconfig resolution logic
- logging/          : klog context wrapper
- version/          : Version information output
- tests/            : E2E tests (using kind cluster)
- hack/             : Development scripts
- bin/              : Build artifacts
- .github/workflows/: CI: test.yml / release.yml

---

## Key Components

### lease package

Core logic that wraps the Kubernetes leader election library (`k8s.io/client-go/tools/leaderelection`).

Key symbols:
- Locker: Locker struct holding namespace/name/id
- NewLocker: Constructor for Locker; accepts various options
- LockAndRun: Main flow of leader election → function execution → cleanup
- ErrElectTimedOut: Sentinel error returned when leader election times out
- CommonLabels(): Returns the label app.kubernetes.io/managed-by=k8s-lease-klock

LockAndRun flow:
1. Attempt leader election via the Lease resource
2. If leadership is not acquired within LeaderElectTimeout (0 = unlimited), return ErrElectTimedOut
3. Once leadership is acquired, invoke f(ctx)
4. If CleanupLease=true, delete the Lease after processing

### process package

Process runs a command under lock control using Locker.

- CancelSignal: Signal sent to the child process when the lock is released or timed out (default: SIGTERM)
- WaitDelay: Grace period before sending SIGKILL after the initial signal (--kill-after)

### kconfig package

kubeconfig resolution priority:
1. In-cluster config (Pod environment variables)
2. --kubeconfig flag
3. $KUBECONFIG environment variable
4. ~/.kube/config

---

## klock Command

### Basic Syntax

  klock [flags] -- <command> [arguments]

Specify the command to execute after --.

### Key Flags

  -l, --lease string              Name of the Lease resource (default: "klock")
  -n, --namespace string          Namespace of the Lease (default: "default")
  -i, --identity string           Holder identity ID (default: "klock")
  -g, --generate-identity         Generate a UUID as the holder identity
  -w, --wait duration             Timeout for acquiring the lock (0 = unlimited)
      --timeout duration          Alias for --wait
  -E, --conflict-exit-code uint8  Exit code used when --wait timeout is reached (default: 1)
      --cleanup-lease             Delete the Lease after processing
  -u, --unlock                    Alias for --cleanup-lease
  -s, --signal value              Signal to send on cancel (default: TERM)
  -k, --kill-after duration       Time to wait before sending SIGKILL after initial signal
      --lease-duration duration   Total time a leader holds the lock before it expires (default: 15s)
      --renew-deadline duration   Time limit for the leader to renew its lock (default: 10s)
      --retry-period duration     Interval between lock acquisition/renewal attempts (default: 2s)
      --labels value              Additional labels to apply to the Lease
      --kubeconfig string         Path to kubeconfig file
  -V, --version                   Display version and exit

### Examples

  # Run some_cmd exclusively using a random UUID as the holder identity
  klock -l some_cmd_lease -g -- some_cmd

  # Use an explicit identity
  klock -l some_cmd_lease -i "$(uuidgen)" -- some_cmd

  # Exit with code 2 if the lock cannot be acquired within 30 seconds
  klock -l some_cmd_lease -g -w 30s -E 2 -- some_cmd

  # Delete the Lease after execution
  klock -l some_cmd_lease -g --cleanup-lease -- some_cmd

### Exit Codes

- 0: Success (command exited with code 0)
- 1: klock error (flag parsing failure, Kubernetes connection failure, etc.)
- Value of -E: Lock acquisition failed due to --wait timeout
- Command's exit code: The executed command failed after the lock was acquired

### Required RBAC Permissions

  apiVersion: rbac.authorization.k8s.io/v1
  kind: Role
  metadata:
    name: klock-role
  rules:
    - apiGroups: ["coordination.k8s.io"]
      resources: ["leases"]
      verbs: ["create", "get", "update", "patch"]
      # Add "delete" if using --cleanup-lease

---

## Development Guide

### Prerequisites

- Go v1.26.5+
- Docker v28.5.1+
- direnv (https://github.com/direnv/direnv)
- Run make init to generate .env.github

### Common Commands

  # Build
  make bin/klock

  # Run all tests (automatically sets up kind cluster)
  make test

  # Unit tests only
  make test-unit

  # E2E tests only
  make test-e2e

  # Lint (vet + golangci-lint + license check)
  make lint

  # Code generation (config_generated.go, etc.)
  make generate

  # Update third-party licenses
  make NOTICE

### Test Strategy

- lease/lease_test.go: Integration tests for Locker using Ginkgo/Gomega (parallel execution, timeout, cleanup)
- process/signal_test.go: Unit tests for signal conversion using testify
- tests/: E2E tests for the full klock command on a kind cluster using testify

Tests require a kind cluster (hack/setup-cluster.sh).

### Code Generation

lease/config_generated.go is auto-generated by goconfig (github.com/berquerant/goconfig).
The go:generate directive is at the top of lease/lease.go.

  make generate
  # or
  go generate ./...

### CI/CD

- .github/workflows/test.yml   : Runs make lint + make test on Push/PR
- .github/workflows/release.yml: Releases binaries via goreleaser on tag Push

---

## Key Dependencies

- k8s.io/client-go              : Kubernetes API client and leader election
- k8s.io/apimachinery           : Kubernetes object metadata types
- github.com/spf13/pflag        : CLI flag parsing
- github.com/google/uuid        : UUID generation for holder identity
- golang.org/x/sys/unix         : Signal name/number conversion
- al.essio.dev/pkg/shellescape  : Shell escaping for command arguments
- k8s.io/klog/v2                : Structured logging
- github.com/onsi/ginkgo/v2     : BDD testing framework
- sigs.k8s.io/controller-runtime: Kubernetes client for testing
- github.com/berquerant/goconfig: Code generation (Config/Builder pattern)

---
> Source: [berquerant/k8s-lease](https://github.com/berquerant/k8s-lease) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-13 -->
