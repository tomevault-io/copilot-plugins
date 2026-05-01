## color-system

> melta UI color token reference


# melta UI Color System

## Primary Palette
| Token | Value | Tailwind |
|-------|-------|----------|
| primary-50 | #f0f5ff | `bg-primary-50` / `text-primary-50` |
| primary-100 | #dde8ff | `bg-primary-100` |
| primary-200 | #c0d4ff | `bg-primary-200` |
| primary-300 | #95b6ff | `text-primary-300` (dark link) |
| primary-400 | #6492ff | `text-primary-400` (dark accent) |
| primary-500 | #2b70ef | `bg-primary-500` |
| primary-600 | #2250df | `bg-primary-600` (hover) / `text-primary-600` |
| primary-700 | #1a40b5 | `hover:bg-primary-700` |
| primary-800 | #13318d | `bg-primary-800` |
| primary-900 | #0e266a | `bg-primary-900` |
| primary-950 | #07194e | `bg-primary-950` |

## Text Colors (3 Tiers)
| Tier | Color | Tailwind | Usage |
|------|-------|----------|-------|
| Primary | #0f172a | `text-slate-900` | Headings |
| Secondary | #3d4b5f | `text-body` | Body text |
| Tertiary | #64748b | `text-slate-500` | Muted/captions |

## Status Colors
| Status | Background | Text | Border |
|--------|-----------|------|--------|
| Success | `bg-emerald-50` | `text-emerald-700` | `border-emerald-200` |
| Warning | `bg-amber-50` | `text-amber-700` | `border-amber-200` |
| Danger | `bg-red-50` | `text-red-700` | `border-red-200` |
| Info | `bg-primary-50` | `text-primary-800` | `border-primary-200` |

## Semantic CSS Variables
```css
:root {
  --bg-page: #f9fafb;      --bg-surface: #ffffff;
  --text-heading: #0f172a;  --text-default: #3d4b5f;
  --text-muted: #64748b;    --border-default: #e2e8f0;
}
```

## Do NOT use
- `text-black`, `bg-gray-300`+, `border-gray-100`
- `bg-green-*`, `bg-yellow-*`, `bg-rose-*`, `bg-blue-*`, `bg-indigo-*`

---
> Source: [tsubotax/melta-ui](https://github.com/tsubotax/melta-ui) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-21 -->
