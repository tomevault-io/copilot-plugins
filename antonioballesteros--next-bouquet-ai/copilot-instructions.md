## shadcn-add

> How to add new shadcn/ui components — use the CLI with network access


# Adding shadcn/ui components

## Rule

**Always add new shadcn components via the official CLI:**

```bash
npx shadcn@latest add <component-name>
```

Examples: `npx shadcn@latest add textarea`, `npx shadcn@latest add card`.

Do **not** create shadcn-style components by hand (e.g. copying from docs) when the component exists in the registry. Use the CLI so the project stays aligned with the registry (style: radix-nova, aliases, dependencies).

## Why the CLI can fail: "You are not authorized to access the item"

This error appears when the CLI tries to fetch the component from `https://ui.shadcn.com/r/styles/radix-nova/<component>.json` and the request fails. Common causes:

1. **Sandbox / restricted network**  
   If the command runs in an environment with limited or blocked outbound access (e.g. Cursor’s sandbox, CI, or “allowlist only” network), the request to `ui.shadcn.com` can be blocked or get a non-200 response, and the CLI may report it as “not authorized”.

2. **Corporate proxy or firewall**  
   Proxies or SSL inspection can alter or block requests to the registry and produce similar errors.

3. **VPN or regional blocking**  
   Some networks or regions may block or rate-limit access to the registry.

4. **Temporary registry issue**  
   The official registry can occasionally return 403/503; retrying later or from a different network often fixes it.

## How to avoid the error

- Run `npx shadcn@latest add <component>` in an environment with **full outbound internet access** (e.g. your local terminal, not a restricted sandbox).
- If you run it via an agent or automated tool, ensure the command is allowed to access the network (e.g. request `network` or `full_network` permissions if the runner restricts them).
- If it still fails, try from your machine in a normal terminal; if it works there, the earlier failure was likely due to the execution environment (sandbox/network).

## If the CLI cannot be used

Only if the CLI genuinely cannot be used (e.g. registry down, no network at all): add the component manually from the [shadcn docs](https://ui.shadcn.com/docs/components) and match the project’s existing style (e.g. `components.json` style and aliases). Prefer fixing the environment and using the CLI when possible.

---
> Source: [antonioballesteros/next-bouquet-ai](https://github.com/antonioballesteros/next-bouquet-ai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-05 -->
