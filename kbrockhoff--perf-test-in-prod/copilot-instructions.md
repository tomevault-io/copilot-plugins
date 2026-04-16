## perf-test-in-prod

> Leveraging the Universal Scalability Law

# Performance Testing in Production

Leveraging the Universal Scalability Law

## Purpose

This repository supports the talk of the same name. The talk explains and illustrates how system
performance can be tested and improved without having to stand up a test environment similar to production.

## Technologies

System under test is a Spring Boot app which returns a generated Atom feed of Lorem Ipsum with artificially
induced delays to simulate a microservices architecture. The system under test runs on a Kubernetes cluster
with Prometheus collecting metrics backed by long-term storage provided by InfluxDB. A namespace with a
Apache JMeter cluster with test results stored in InfluxDB is used to put load on the system.

R and its USL library are used to calculate the Universal Scalability Law equations from the
resulting metrics.

## Results

Prometheus Query: http_server_requests_seconds{kubernetes_name="aggregate-svc",uri="/feeds/{feedId}"}

---
Source: .ruler/java-git.md
---
# AI Agent Git Operations for Java Repositories

## Core Principles

**Simplicity First**: Each operation should have a single, clear purpose. Avoid complecting git operations with unrelated concerns.

**Composable Workflow**: Each stage builds independently on the previous, allowing for easy debugging and iteration.

## Branch Management

### Creating Branches
- **Format**: `feature/descriptive-name` or `bugfix/issue-description`
- **Source**: Always branch from `main`
- **Naming**: Use kebab-case, be specific about the change

```bash
git checkout main
git pull origin main
git checkout -b feature/add-vpc-module
```

### Keeping Branches Current
- **Method**: Rebase only (preserve linear history)
- **Frequency**: Daily, or before any major work session
- **Conflict Resolution**: Address conflicts immediately

```bash
git fetch origin
git rebase origin/main
```

## Pre-Commit Standards

### Validation Pipeline (Required)
Execute only in modules with changes:

1. **Maven Build (If pom.xml exists)**
   ```bash
   mvn -B verify
   ```

2. **Gradle Build (If build.gradle exists)**
   ```bash
   ./gradlew build
   ```

## Commit Standards

### Message Format
Follow [Conventional Commits 1.0.0](https://www.conventionalcommits.org/):

```
<type>[optional scope]: <description>

[optional body]

[optional footer(s)]
```

### Examples
```
feat(vpc): add multi-az subnet configuration
fix(security-group): resolve circular dependency issue
docs(readme): update module usage examples
test(vpc): add integration test for NAT gateway
```

### Commit Scope
- **Atomic Changes**: One logical change per commit
- **Complete Units**: Each commit should leave the repository in a working state
- **Clear Intent**: Commit message should explain *why*, not just *what*

## Pull Request Workflow

### PR Requirements
1. **Title**: Must follow Conventional Commits format
2. **Description**: Complete all GitHub template sections
3. **Scope**: Single concern per PR (avoid complecting multiple features)
4. **Size**: Prefer smaller, focused PRs over large, complex ones

### PR Template Sections (All Required)
- [ ] **Summary**: What does this PR accomplish?
- [ ] **Changes**: Specific modifications made
- [ ] **Testing**: How was this validated?
- [ ] **Breaking Changes**: Any backwards compatibility concerns?
- [ ] **Documentation**: What docs were updated?

## Quality Gates

### Automated Checks (Must Pass)
- Maven or Gradle build

### Manual Review Focus
- **Simplicity**: Is the solution as simple as possible?
- **Composition**: Are components properly decoupled?
- **Clarity**: Is the intent obvious from the code?
- **Testability**: Can changes be easily validated?

## Troubleshooting

### Common Issues
1. **Rebase Conflicts**: Resolve immediately, don't accumulate
2. **Test Failures**: Fix root cause, don't mask symptoms
3. **Lint Errors**: Address policy violations, don't suppress

### Recovery Patterns
```bash
# Reset to clean state if needed
git reset --hard origin/main

# Clean workspace
git clean -fd

```

## Notes for AI Agents

- **Fail Fast**: Stop pipeline on first error
- **Context Preservation**: Maintain clear audit trail of operations
- **Rollback Strategy**: Always know how to undo changes

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/kbrockhoff) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
