## unity

> Prefer the full local Unity stack for internal development


# Full Local Stack First

This is internal agent/developer guidance. It intentionally differs from the
public `unity` README: the README is for open-source users who run `unity`
against hosted Orchestra and do not have the private `unity-deploy` repo. For
internal cross-repo work, default to the private full local stack in
`unity-deploy/selfhost/`.

Before starting, stopping, rebuilding, or repairing any local service, assume the
full stack may already be running and inspect it first:

```bash
bash /Users/djl11/unity-deploy/selfhost/stack.sh status
```

Default commands:

- Start full source stack: `bash /Users/djl11/unity-deploy/selfhost/stack.sh up`
- Repair only Console: `bash /Users/djl11/unity-deploy/selfhost/stack.sh repair-console`
- Smoke test: `bash /Users/djl11/unity-deploy/selfhost/stack.sh smoke`

Do not run isolated commands against a live stack unless the user explicitly asks
for isolated mode: `npm run dev`, `next dev`, `next build`, `npm run ci`,
`console/scripts/local.sh start`, `orchestra/scripts/local.sh start`, or
`unity/scripts/local.sh start`.

If isolated mode is explicitly requested, use the relevant override env var and
explain which full-stack guarantees are being bypassed.

---
> Source: [unifyai/unity](https://github.com/unifyai/unity) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-27 -->
