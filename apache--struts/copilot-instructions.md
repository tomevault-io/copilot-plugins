## struts

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

For detailed procedures, use the specialized agents and commands in `.claude/agents/` and `.claude/commands/`.

## Project Overview

Apache Struts is a mature MVC web application framework for Java (originally WebWork 2). Current version: **7.2.0-SNAPSHOT**. Uses OGNL for value stack expressions and FreeMarker for UI tag templates.

### Build Commands

```bash
# Run tests (skip assembly for speed)
mvn test -DskipAssembly

# Single test in specific module
mvn test -DskipAssembly -pl core -Dtest=MyClassTest#testMethodName

# Jakarta EE 11 / Spring 7 profile
mvn clean install -Pjakartaee11
```

### Project Structure

```
struts/
├── core/           # struts2-core - main framework
├── plugins/        # Plugin modules (json, rest, spring, tiles, velocity, etc.)
├── apps/           # Sample applications (showcase, rest-showcase)
├── assembly/       # Distribution packaging
├── bom/            # Bill of Materials for dependency management
├── parent/         # Parent POM with shared configuration
└── jakarta/        # Jakarta EE compatibility modules
```

### Core Architecture

**Request Lifecycle**: `Dispatcher` → `ActionProxy` → `ActionInvocation` → Interceptor stack → `Action` → Result

Key packages in `org.apache.struts2`:

- `dispatcher` - Request handling, `Dispatcher`, servlet integration
- `interceptor` - Built-in interceptors (params, validation, fileUpload)
- `components` - UI tag components (form, textfield, submit)
- `action` - Action interfaces (`UploadedFilesAware`, `SessionAware`, etc.)
- `security` - Security utilities and OGNL member access policies

## Security-Critical Patterns

Apache Struts has a history of security vulnerabilities (OGNL injection, temp file exploits). Apply these Struts-specific patterns:

1. **Temporary files**: Use UUID-based names in controlled locations (see example below)
2. **OGNL expressions**: Evaluate only framework-generated OGNL; use allowlist member access
3. **File uploads**: Validate content types, sanitize filenames, enforce size limits
4. **Parameter filtering**: Use `ParameterNameAware` to restrict accepted parameter names

```java
// Secure temporary file pattern
protected File createTemporaryFile(String fileName, Path location) {
    String uid = UUID.randomUUID().toString().replace("-", "_");
    return location.resolve("upload_" + uid + ".tmp").toFile();
}
```

## Security Reports & Scans

For any security-related activity — vulnerability scans, security analysis, drafting security reports — **[`SECURITY.md`](SECURITY.md) is the source of truth**.
Read it first and follow its pre-reporting checks, assessment checklist, and reporting requirements. Reports must be sent privately to
`security@struts.apache.org`; do not open a public GitHub issue, Jira issue, pull request, or mailing list thread for a suspected vulnerability before private
triage. [`AGENTS.md`](AGENTS.md) is a shorter LLM-facing wrapper around the same process.

## Testing

Tests use JUnit 5 with AssertJ assertions and Mockito for mocking. Run with `mvn test -DskipAssembly`.

## Pull Requests

- **Title format**: `WW-XXXX Description` (Jira ticket ID required)
- **Link ticket in description**: `Fixes [WW-XXXX](https://issues.apache.org/jira/browse/WW-XXXX)`
- **Issue tracker**: https://issues.apache.org/jira/projects/WW
- **Never submit a PR that fixes a suspected vulnerability.** Before opening a PR, verify the change is not a security patch (OGNL injection, parameter
  filtering bypass, file upload exploit, auth bypass, RCE, SSRF, path traversal, deserialization, XSS in framework components, etc.). If it is, stop and report
  it privately to `security@struts.apache.org` — see [`SECURITY.md`](SECURITY.md).

---
> Source: [apache/struts](https://github.com/apache/struts) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-23 -->
