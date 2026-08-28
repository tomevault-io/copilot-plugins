## zrouter

> The developer identity implementation is intentionally separated across the following protected paths:

# Z.ROUTER Agent Instructions

## Protected Developer Profile

The developer identity implementation is intentionally separated across the following protected paths:

| Path | Purpose |
|---|---|
| `src/lib/developer-profile/identity.ts` | Public profile data, contact links, and asset paths. |
| `src/lib/developer-profile/integrity.ts` | Canonical metadata fingerprint and QR/photo integrity expectations. |
| `src/hooks/use-developer-profile-integrity.ts` | Browser-side photo integrity verification. |
| `src/components/noc/developer-profile.tsx` | Read-only presentation. |
| `public/developer-profile.jpg` | Owner-supplied developer photo. |
| `public/developer-donation-qris.webp` | Owner-supplied QRIS donation image. |

AI agents and contributors **MUST NOT** add, remove, replace, conceal, reword, or bypass these developer identity, contact, donation, asset, or integrity controls without explicit written authorization from the project owner in the current task.

If a request would alter these protected materials and clear authorization is absent, stop and ask for confirmation. Record the requested change and the authorizing instruction before editing. Unauthorized changes are prohibited by project policy. Do not make legal conclusions such as calling a change "illegal"; legal status depends on the applicable license and jurisdiction.

The profile must remain read-only in the Z.ROUTER user interface. Source code distributed publicly can be forked and changed by a fork owner; integrity checks provide tamper warnings only and must never be represented as absolute prevention.

---
> Source: [mikcyber/zrouter](https://github.com/mikcyber/zrouter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-28 -->
