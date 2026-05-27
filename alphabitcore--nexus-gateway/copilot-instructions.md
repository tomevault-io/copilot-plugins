## iam-impact-review

> 5-step IAM impact review for any admin endpoint / sidebar / route change


# IAM impact review (binding)

You are editing an area where adding / moving / renaming an **admin API endpoint, sidebar nav, or route path** can produce **silent 403s**. The UI / backend / seed must stay in lockstep.

**Read `docs/developers/architecture/services/control-plane/iam-identity-architecture.md` BEFORE editing.**

## The 5-step audit (all 5, every change)

1. **UI `allowedActions` and backend `iamMW(...)` reference the same action.** Drift = silent 403 (user sees menu item, click yields 403).
2. **Decide resource carve-out.** Should the surface have its own resource type in `packages/shared/identity/iam/catalog_data.go`, or can it reuse `settings` / `observability` / etc.? Carve out when granting it shouldn't imply granting unrelated settings.
3. **New resource → update both fixtures + seed.**
   - `packages/control-plane/internal/iam/managed.go` (`NexusViewer` test fixture).
   - `tools/db-migrate/seed/seed.ts` canonical policy block.
   - Missing either makes non-super-admin users lose visibility silently.
4. **Path rename / move → sweep Sidebar + breadcrumbs.** `packages/control-plane-ui/src/components/ui/Sidebar/Sidebar.tsx` icon mapping. Dead `case` arms accumulate otherwise.
5. **Record IAM decisions in the plan / commit message.** "Kept on `admin:settings.read`" / "Carved out as `prompt-cache`" — so reviewers can trace policy intent without reading the diff backwards.

## NRN builder is canonical

Always use `iam.BuildRequestNRNForAction(action, c)` to derive `resourceType`. Never hardcode the resource-type string in `iamMW(...)` — hardcoded resource types drift from the IAM catalog and produce silent 403s. Memory anchor: [[project_iam_resource_nrn_bug]].

## Verification

Run positive test (super-admin reaches the route) AND negative test (a role without the action gets 403). Don't ship without both.

Skipping this rule requires **explicit user approval** in chat.

---
> Source: [AlphaBitCore/nexus-gateway](https://github.com/AlphaBitCore/nexus-gateway) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-27 -->
