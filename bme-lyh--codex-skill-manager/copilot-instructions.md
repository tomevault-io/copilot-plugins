## codex-skill-manager

> This repository builds Codex Skill Manager for Windows. Preserve the security model and transaction semantics.

# Agent Instructions

## Scope

This repository builds Codex Skill Manager for Windows. Preserve the security model and transaction semantics.

## Safety invariants

- Never modify or install into `.codex/skills/.system`.
- Never add recursive, wildcard, piped, looped, or multi-target permanent deletion.
- `remove` means moving each explicit skill directory to quarantine.
- Do not execute scripts obtained from skill repositories.
- Resolve GitHub branches and tags to immutable commit SHAs before planning installation.
- Never log tokens, cookies, authorization headers, or credential helper output.
- Treat all downloaded skill content as untrusted.
- Cloud or LLM scanning must remain opt-in.
- Preserve local modifications unless the user explicitly approves replacement.

## Development workflow

1. Run `gofmt` on changed Go files.
2. Run `go test ./...`.
3. Run TypeScript type checking and the Vite production build after frontend changes.
4. Build the desktop executable with `wails build`; plain `go build` omits the
   required Wails build tags and produces a documentation warning at runtime.
5. Validate `skills/codex-skill-manager` with the bundled skill validator after changing it.
6. Update user and agent documentation when a command contract or data schema changes.

## Completion bar

A mutating feature is incomplete until it has:

- a dry-run or plan surface;
- an explicit target;
- backup or quarantine behavior;
- transaction journaling;
- structured JSON output;
- failure reporting and a recovery path.

---
> Source: [bme-lyh/Codex-Skill-Manager](https://github.com/bme-lyh/Codex-Skill-Manager) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-03 -->
