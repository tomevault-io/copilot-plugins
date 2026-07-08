## edictus-sns

> Handles Notification, SubscriptionConfirmation, and UnsubscribeConfirmation message types.

# @jogi/sns — AWS SNS Signature Verification

Cryptographic verification of AWS SNS messages per the AWS SNS signature spec.
Handles Notification, SubscriptionConfirmation, and UnsubscribeConfirmation message types.
Extracted from [jogi](../jogi) to be reusable across projects that receive SNS webhooks.

## Compact Instructions

When compacting, preserve: file paths changed, errors found, decisions made. Drop: full file contents already read, tool output bodies.

## Communication Style

- **No emotional validation** — never say "I understand your frustration". Results matter, not words.
- **No excessive apologies** — don't apologize repeatedly. Fix the problem.
- **Be direct** — state facts, propose solutions, execute. Skip the fluff.
- **Ask for input** — when stuck or facing multiple approaches, ask rather than guessing.

## Tech Stack

- **Runtime**: Node.js (built-in `crypto`, no external deps)
- **Build**: tsup (ESM + CJS + types)
- **Tests**: vitest

## Project Structure

```
src/
└── index.ts    # All exports — verifySNSMessage, verifySNSSignature, validateTopicArn, types
```

## Code Rules

1. **No external dependencies** — uses only Node.js built-ins (`crypto`, `fetch`)
2. **Security-first** — never weaken certificate validation. Cert URL must be HTTPS from `amazonaws.com`. Cert must be issued by Amazon and not expired.
3. **In-memory cert cache** — 1-hour TTL, keyed by URL. Do not persist to disk.
4. **Both signature versions** — support SignatureVersion 1 (SHA1) and 2 (SHA256)
5. **API stability** — exported interfaces (`SNSMessage`, `VerifySNSOptions`) must stay backward-compatible with jogi. Breaking changes require updating jogi's SNS handler
6. **Test coverage** — after implementing a feature, check if tests exist. Update or write tests. Never leave a feature without test coverage.
7. **Planning** — for non-trivial changes, write a plan to `docs/plans/` before implementing

## Commands

```bash
npm run build        # Build dist/ (ESM + CJS + types)
npm run dev          # Build in watch mode
npm test             # Run unit tests
npm run test:watch   # Watch mode
```

## Validation

Use `npx tsc --noEmit` for fast type checking. Run `npm run build` to verify bundling. Run `npm test` before committing.

## Consumer Integration

Consumed by jogi via GitHub reference:
```json
"@jogi/sns": "github:luvidal/jogi-sns#main"
```

Primary consumer: `~/GitHub/jogi` — see its `pages/api/` SNS webhook handler for usage context.

---
> Source: [luvidal/edictus-sns](https://github.com/luvidal/edictus-sns) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
