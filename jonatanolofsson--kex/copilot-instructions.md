## kex

> Annotation-driven cluster landing page + per-app detail view for ArgoCD-managed clusters.

# kex

Annotation-driven cluster landing page + per-app detail view for ArgoCD-managed clusters.

## Quick reference

```bash
direnv allow            # activate the nix flake dev shell
just                    # list recipes
just check              # lint + fmt-check + pytest + vitest + docs build
just dev                # FastAPI on :8000 with auto-reload
just ui-dev             # Vite/Svelte on :5173 (proxies /api to :8000)
just build              # local docker build (CI pushes to ghcr.io on tag)
```

All commands need the nix dev shell. If direnv hasn't loaded, prefix with `direnv exec . ...`.

## Layout

| Path | Contents |
|---|---|
| `src/kex/` | FastAPI backend |
| `ui/` | Svelte 5 + Vite SPA |
| `chart/` | Helm chart (deployed via ArgoCD) |
| `docs/` | Sphinx docs |

## Design invariants (don't break these)

1. **Annotation-driven, never code-driven.** New field → new `kex/*` annotation, never a switch on app name.
2. **All external data sources cached + gracefully degradable.** ArgoCD down → page renders without git column, not 500.
3. **SPA + backend separation.** Frontend only talks to `/api/*` JSON.
4. **Argo as the integration point, not git directly.** Never introduce a second credential channel.
5. **RBAC is strictly read-only.** Write actions need a separate threat-model review.

See `docs/design-principles.rst` for the full list.

## Workflow

`docs → test → implement → check` before commit. Conventional commits (`feat:`, `fix:`, `docs:`, …).

## Advising consumers (apps registering on the index)

Apps surface themselves on kex by adding `kex/*` annotations to their
own ArgoCD `Application` CR — no central catalog edit.

- **Required:** `kex/enabled: "true"`.
- **Strongly recommended:** `kex/title`, `kex/description`, `kex/group`.
- **Optional:** `kex/icon`, `kex/about` (markdown for the detail page),
  `kex/links.<label>: <url>`, `kex/groupweight` / `kex/weight` (ordering).

Kubernetes annotation values are always strings — booleans and integers must be quoted
(`"true"`, `"-30"`). Validate at the consumer's generator layer so a TOML `true`
doesn't silently land as a number.

For apps whose Application manifest is **auto-generated** (e.g. ruskin's
`rollout-argocd-gen` used by truckbed and converter-ir; helm chart values for
self-hosted apps), the annotations must come from the generator's input,
not from hand-edits to the generated YAML — otherwise the next
regeneration wipes them. Ruskin exposes a `[argocd.annotations]` table in
`rollout.toml` for exactly this; see `truckbed/rollouts/aitik/rollout.toml`
for a worked Truckbed example.

`docs/annotations.rst` is the canonical contract — the cluster-wide
`groupweight` convention table lives there. Point consumers at that file
rather than restating the conventions ad-hoc; keeping the convention
single-sourced prevents drift between docs and reality.

---
> Source: [jonatanolofsson/kex](https://github.com/jonatanolofsson/kex) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-21 -->
