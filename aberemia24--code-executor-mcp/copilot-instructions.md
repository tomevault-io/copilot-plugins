## code-executor-mcp

> > 📚 **Quick Reference:** Type these in chat to load into context:


# Claude Instructions for code-executor-mcp

> 📚 **Quick Reference:** Type these in chat to load into context:
> - `@docs/coding-standards.md` - SOLID/DRY/KISS, TDD, best practices
> - `@docs/release-workflow.md` - Patch/minor/major release steps

## 🚨 CRITICAL: Always Use Code Executor MCP

**MANDATORY:** Use `mcp__code-executor__executeTypescript` + `callMCPTool` for ALL operations:
- ❌ **DON'T:** Write tool, Read tool, Bash commands for file operations
- ✅ **DO:** `executeTypescript` with `callMCPTool('mcp__filesystem__write_file', ...)`

**Why this matters:**
- Single round-trip (discover + execute + verify in one call)
- Tests the actual MCP we're building (dogfooding)
- Variables persist across operations (no context switching)
- Real-world usage pattern that validates our architecture

**Example - File Operations:**
```typescript
// ❌ BAD: Using traditional tools
Write('/tmp/test.json', content);  // Doesn't test our MCP

// ✅ GOOD: Using code-executor MCP
await mcp__code-executor__executeTypescript({
  code: `
    const tools = await discoverMCPTools({ search: ['file'] });
    const content = JSON.stringify({ test: true }, null, 2);
    await callMCPTool('mcp__filesystem__write_file', {
      path: '/tmp/test.json',
      content
    });
    const result = await callMCPTool('mcp__filesystem__read_file', {
      path: '/tmp/test.json'
    });
    console.log('Verified:', JSON.parse(result.content));
  `,
  allowedTools: ['mcp__filesystem__*']
});
```

**When to use traditional tools:**
- Reading project source code for review/analysis
- Git operations (commits, merges, branches)
- Build/test commands (`npm run build`, `npm test`)
- Everything else: Use code-executor MCP

## Project Overview

**code-executor-mcp** - Universal MCP server with progressive disclosure | **98% token reduction** (141k → 1.6k)

**Core Concept:** 2 execution tools (`executeTypescript`, `executePython`) call other MCPs on-demand via `callMCPTool('mcp__server__tool', params)`

**Key Features:** Progressive disclosure | AJV schema validation | AsyncLock schema cache | Deno sandbox | Multi-transport (STDIO/HTTP)

## Current State

**Version:** v0.3.1 (pre-1.0 beta) | **Branches:** `main` (stable, PR-only) + `develop` (active) | **Stack:** TypeScript 5.x + Node.js 20+ + @modelcontextprotocol/sdk + AJV + async-lock + Vitest + Deno

**Recent:** Deep validation (AJV) | AsyncLock mutex | 253 tests (98%+ coverage) | Runtime validation primary approach

## Architecture

**Components:** MCP Proxy Server | MCP Client Pool (STDIO/HTTP) | Schema Cache (24h TTL, AsyncLock) | Schema Validator (AJV) | Executors (TypeScript/Deno, Python)

**Key Files:** `package.json` | `CHANGELOG.md` | `RELEASE.md` | `SECURITY.md`

## Development Workflow

**Branch Strategy:** Work on `develop` → PR to `main` → `npm version` → `gh release create` → sync `develop`

**Commands:** `npm test` | `npm run typecheck` | `npm run build` | `npm run lint`

**Standards:** TDD mandatory | 98%+ coverage (validation/caching) | TypeScript strict | SOLID principles | Security first

**Important:** When performing these tasks, reference the relevant docs:
- **Writing code?** Reference @docs/coding-standards.md for SOLID/DRY/KISS principles, TDD requirements
- **Creating release?** Reference @docs/release-workflow.md for step-by-step patch/minor/major instructions

## Key Decisions

**AJV:** Industry-standard | Deep recursive validation | Self-documenting errors | Zero maintenance
**AsyncLock:** Prevents race conditions | Thread-safe cache writes | Production-ready
**24h TTL:** Schemas rarely change | Reduces network overhead | Stale-on-error resilience

## Common Tasks

**Feature:** `develop` branch → TDD → implement → tests → CHANGELOG → commit → PR
**Bugfix:** Failing test → fix → verify → CHANGELOG → `fix:` commit
**Release:** See [Release Workflow](docs/release-workflow.md) for step-by-step instructions (patch/minor/major)

## Testing

**Structure:** Vitest + TypeScript | Mock dependencies | `vi.useFakeTimers()` | Test edge cases
**Coverage:** Validation 98%+ | Caching 70%+ | Overall 90%+
**Focus:** ✅ Logic/errors/edge cases/security | ❌ Third-party libs

## Security (ZERO TOLERANCE)

**Validation:** MUST validate all MCP tool calls | Nested objects/arrays recursive | No type coercion | No info leakage
**Sandbox:** Minimal Deno permissions | Block eval/exec/__import__ | Prevent path traversal | Rate limiting
**Audit:** Log all executions (timestamp, tool, params hash, status) | NO sensitive data

## Dependencies

**Production:** @modelcontextprotocol/sdk | ajv ^8.17.1 | async-lock ^1.4.1 | zod | ws
**Development:** vitest | typescript | @types/async-lock

## Troubleshooting

**Fake Timers:** `vi.useFakeTimers()` in `beforeEach` | `vi.advanceTimersByTime()` | `vi.useRealTimers()` in `afterEach`
**Cache Corruption:** Check AsyncLock | Delete `~/.code-executor/schema-cache.json`
**Validation:** Check AJV errors | Verify schema | Test minimal params first

## Available Agents (Use Proactively)

- **code-guardian** - Review code quality, SOLID principles, MCP patterns, security (use after implementation)
- **inquisitor** - Debug complex issues, trace root causes, systematic investigation (use for bugs)
- **project-librarian** - Explore codebase, find files/functions, understand structure (use before changes)
- **project-documentarian** - Maintain devlogs, preserve context, JSDoc enhancement (use for documentation)
- **document-reviewer** - Review documentation quality and completeness (use for docs)
- **research-specialist** - Fetch latest library docs, research technical questions (use for unknowns)

## Available Slash Commands (Use Proactively)

- **/build** - Build with TypeScript/ESLint enforcement, clean dist/ artifacts
- **/code-review** - Comprehensive review against MCP server standards, invoke code-guardian
- **/commit** - Create proper git commits with validation, handle pre-commit hooks
- **/debug** - Investigate MCP server issues, schema validation, concurrency problems
- **/fix** - Fix issues at root cause, enforce proper solutions (no quick hacks)
- **/test** - Execute Vitest tests, focus on validation/caching/security coverage
- **/compact_FILE** - Consolidate verbose files, remove duplicates, preserve all info
- **/split-context** - Extract area-specific content into local CLAUDE.md files

## Contact

**Issues:** https://github.com/aberemia24/code-executor-MCP/issues | **Email:** aberemia@gmail.com | **Docs:** https://github.com/aberemia24/code-executor-MCP#readme

---
> Source: [aberemia24/code-executor-MCP](https://github.com/aberemia24/code-executor-MCP) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-29 -->
