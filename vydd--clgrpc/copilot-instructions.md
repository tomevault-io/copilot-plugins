## clgrpc

> - **Never clear the FASL cache** unless it's truly the last resort after exhausting all other debugging options. ASDF handles recompilation automatically when source files change.

# CLAUDE.md - Project Guidelines for Claude Code

## Common Lisp / ASDF

- **Never clear the FASL cache** unless it's truly the last resort after exhausting all other debugging options. ASDF handles recompilation automatically when source files change.

## Running Tests

```bash
sbcl --non-interactive \
  --eval '(push (truename ".") asdf:*central-registry*)' \
  --eval '(asdf:load-system :clgrpc-tests)' \
  --eval '(fiveam:run! :clgrpc-all)'
```

To run a specific test suite:
```bash
sbcl --non-interactive \
  --eval '(push (truename ".") asdf:*central-registry*)' \
  --eval '(asdf:load-system :clgrpc-tests)' \
  --eval '(fiveam:run! :client-tests)'
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/vydd) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
