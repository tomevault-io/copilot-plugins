## keios

> `compose-expert@aldefy-compose-skill` is installed. It is good on Compose the framework and wrong

# KeiOS

## The compose-expert skill: what applies here and what does not

`compose-expert@aldefy-compose-skill` is installed. It is good on Compose the framework and wrong
about this app's UI layer, because it assumes androidx Material3 and androidx Navigation and KeiOS
uses **neither**. Counted in the tree, not assumed:

| | files using it |
|---|---|
| `MiuixTheme` | 357 |
| `MaterialTheme` | **0** |
| `androidx.compose.material3` | **0** |
| `androidx.compose.material.` (M2) | **0** |
| `top.yukonga.miuix.kmp.nav` | 6 |
| androidx Navigation / Navigation 3 | **0** |
| `androidx.paging` | **0** |
| `androidx.tv` | **0** |

(The four files matching `androidx.navigation*` are `androidx.navigationevent`, the predictive-back
event library. That is not Navigation.)

### Reference only — never apply directly

These reference files describe libraries this app does not depend on. Read them for the *idea*;
never port their code, API names, or migration steps into KeiOS.

- `references/navigation.md`, `references/navigation-migration.md`,
  `references/source-code/navigation-source.md`
- `references/theming-material3.md`, `references/material3-motion.md`,
  `references/source-code/material3-source.md`

**The navigation one is the actual trap.** miuix-nav exports `NavDisplay`, `NavKey`,
`NavBackStack` and `rememberNavBackStack` — the *same names* as androidx Navigation 3, from
`top.yukonga.miuix.kmp.nav`. Guidance written for one will look like it compiles against the other
and does not. miuix-nav additionally has `opaqueDepth`, `navSwipeDismiss`, `NavSettleSpec`,
`NavMotion` and `navMaxLifecycleFor`, which androidx has no equivalent of — and issue #21 was
caused by `navSwipeDismiss` specifically. For anything navigation-shaped, read
`app/src/main/java/os/kei/ui/page/main/host/main/MainScreenNavHost.kt` and the miuix-nav sources,
not the skill.

Same shape for theming: KeiOS reads `MiuixTheme.colorScheme` and its own `AppMotionTokens`, so
`MotionScheme`, `MotionTokens` and M3 Expressive components do not exist here.

### Not relevant at all

`paging*.md`, `tv-compose.md`, `multiplatform.md`, `platform-specifics.md`,
`source-code/cmp-source.md`, `styles-experimental.md` — no Paging, no TV, Android-only, no
Compose Multiplatform.

### Genuinely useful

Framework-level and library-agnostic, so it applies unchanged: `state-management.md`,
`side-effects.md`, `performance.md`, `modifiers.md`, `lists-scrolling.md`,
`composition-locals.md`, `animation.md` (the core animation APIs, not the M3 tokens),
`accessibility.md`, `production-crash-playbook.md`, `view-composition.md`, and the
`source-code/` receipts for runtime, ui and foundation.

### Updating

Installed as a plugin, so it updates in place and this file survives:

```bash
claude plugin marketplace update aldefy-compose-skill
```

## Performance work

Frame-time investigations are written up in `docs/planning/` — start with
`hwui-frame-budget.md`, which also records the measurement noise floor and several
already-rejected optimisations. Harness lives in `scripts/perf/`.

---
> Source: [hosizoraru/KeiOS](https://github.com/hosizoraru/KeiOS) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-23 -->
