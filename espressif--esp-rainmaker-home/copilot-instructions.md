## code-standards

> All new and modified code must strictly follow .cursor/rules/code-standards.mdc. Scope: (1) @store only in features/shared—no direct @espressif SDKs. (2) Feature domains vs src/shared UI layout. (3) Flags via features.config.ts + sdk.config.ts SDK caps. (4) Strict TS (SOLID/KISS/YAGNI), mandatory JSDoc, strings in shared/utils/constants.ts.


# Code standards

## 1 — Feature ↔ SDK boundary

**Thumb rule:** **`src/features/`** (and UI-oriented code under **`src/shared/`**) must **not** import or invoke **`@espressif/*-sdk`** packages, SDK singletons, or raw cloud client APIs. The product layer talks to **`@store`** (CDF / ESPCDF entities, MobX stores, `useCDF`, store actions, synchronizers) only.

**Where SDK usage belongs:** **`src/sdk-adaptors/`** (adaptor + transformers), **`src/integrations/`** (bootstrap), and **`config/sdk.config.ts`** wiring—not screens, feature hooks, or shared components.

**Do:** read and mutate state through **store** types and APIs already exposed for the UI.

**Do not:** add `import … from "@espressif/rainmaker-base-sdk"` (or RMNG / Matter equivalents) under **`src/features/`** or **`src/shared/`** unless an explicit exception is documented at the repo level.

If a feature needs new data or an API call, extend the **adaptor** / **store** path first, then consume it from the feature.

---

## 2 — UI structure: feature domains vs shared

Same **split** (presentation / hooks / pure utils / reusable UI), applied in two places: **one domain** under `src/features/<domain>/`, or **every domain** under **`src/shared/`**.

### A — Feature domain (`src/features/<domain>/`)

When adding or changing **screens**, keep the **domain layout**:

| Put it here | Responsibility |
|-------------|----------------|
| **`screens/`** | **Compose** the screen: wire hooks, pass props, layout. Prefer **presentation** (minimal logic in the screen file). **Styling** via **`theme/`** (domain or **`@shared/theme`**) and tokens—avoid inline magic numbers when a token or style exists. |
| **`hooks/`** | **Reusable** stateful logic for **that domain**: effects, `useCDF` / store usage, navigation, form state, derived data shared inside the feature. |
| **`utils/`** | **Pure** helpers only: formatting, mapping, calculations, **no React state**, **no hooks**, **no JSX**. If it needs `useState` / `useEffect`, use a **hook**. |
| **`components/`** | **Reusable UI** for **this domain** only; break large screens into pieces; **props in, elements out**. |

**Flow (domain):** Screen → domain **hooks** → domain **components**; **utils** for pure helpers. Move UI to **`src/shared/`** once **two or more** domains need it.

**Avoid (domain):** thousand-line screens, logic in `utils/`, or stateful behavior stuffed into `components/` when a **hook** is clearer.

### B — Shared (`src/shared/`) — cross-domain reuse

**`src/shared/`** holds building blocks used **across feature domains** (ParamControls, layout, navigation, form primitives, etc.). Apply the **same** discipline, stricter **neutrality**:

| Put it here | Responsibility |
|-------------|----------------|
| **`components/`** | **Domain-agnostic** UI: no imports from **`src/features/<one-domain>/`**. Encode behavior via **props and callbacks**, not feature-specific globals. Split large widgets into smaller shared components (e.g. `ParamControls/`, `Layout/`, `Form/`). |
| **`hooks/`** | **Generic** hooks reusable anywhere: `useCDF`, throttling, toast, debounce—**no** single-feature screen names or copy; keep APIs portable. |
| **`utils/`** | **Pure**, cross-cutting helpers (formatting, validation, constants)—**no** hooks, **no** JSX, **no** feature-only assumptions. |
| **`theme/`** | **Global** tokens and shared stylesheets (`tokens`, `globalStyleSheet`, …)—consumers use tokens instead of one-off literals. |

**Flow (shared):** Features import from **`@shared/...`**; shared **never** depends on a specific **`features/<domain>`** module (avoid circular coupling). If something is only used by one domain, it stays under **`features/<domain>/`**, not shared.

**Avoid (shared):** embedding one product flow inside a shared component; leaking domain copy or navigation tied to a single feature; putting stateful orchestration in `utils/`.

---

## 3 — Modular feature domains and feature flags

Treat each **`src/features/<domain>/`** folder as a **separate product module** where it maps to product capability (schedule, scene, provision, group, …). Visibility and behavior are **not** hard-coded only in UI: they are **controlled together** by **`config/features.config.ts`** and **SDK support** in **`config/sdk.config.ts`** (`SDK_FEATURE_MAP` / `getResolvedActiveSdk`).

**How flags work (two levels):**

1. **SDK capability (hard gate)** — From **`sdk.config`**: if the **active SDK** does not support a capability, that feature stays **off** regardless of env.
2. **Env / `app.config` extra (`features.*`)** — Can **disable** a feature the SDK allows; it **cannot** enable something the SDK blocks.

Use **`getFeatures()`** from **`@config/features.config`** (or the project’s re-export) in hooks/screens to **gate navigation, tabs, and entry points**—avoid scattered ad-hoc checks.

**When adding or extending a togglable capability:**

- Add or update the **`FeatureKey`** / **`ENV_KEY_MAP`** / **`getFeatures()`** resolution in **`features.config.ts`** as needed.
- Ensure **`SDK_FEATURE_MAP`** in **`sdk.config.ts`** reflects each **active SDK** (RM / RMNG / Matter) truthfully.
- Keep **domain code** under the matching **`src/features/<domain>/`** module so the module can be reasoned about and flagged as a unit.

**Avoid:** bypassing `getFeatures()` with raw `Constants.expoConfig.extra` checks everywhere; assuming one SDK—always respect **active SDK** + **SDK map** + **env** together.

---

## 4 — TypeScript style (strict): SOLID, KISS, YAGNI; JSDoc; string constants

**Strictly follow** this section for all new and touched TS/TSX.

### SOLID, KISS, YAGNI

- **SOLID:** One clear responsibility per module; depend on **interfaces/types** at boundaries; extend behavior without editing unrelated code when reasonable.
- **KISS:** Smallest change that satisfies the requirement—no speculative layers.
- **YAGNI:** No extra abstraction, flag, or parameter until there is a **concrete** need.
- **Types:** **`any` is forbidden** unless an unavoidable escape is documented in-place with a **one-line `// eslint-disable` + reason**; prefer **`unknown`** + narrowing. Public APIs **must** be typed. Use **narrow unions** and **`as const`** for fixed string sets where it helps.

### JSDoc (mandatory)

- **Every** `function`, **method**, **custom hook**, and **React component** (including **non-exported** helpers in the same file) **must** start with a **`/** … */`** block describing **what it does** and **why** when behavior is not obvious from the name alone.
- **Exported** functions and hooks **must** include **`@param`** for each parameter and **`@returns`** when the return value is not `void` / `undefined`, unless the signature is trivial getters.
- **React components** **must** document **purpose** and, when non-obvious, **side effects** (navigation, store writes, subscriptions).

### String constants (mandatory)

- **Must not** use **raw string literals** in **`if` / `switch` / comparisons** for **technical** values: SDK/param/type names, status/discriminator strings, route keys, error tags, storage keys, known API constants, etc.
- **Must** define them as **`export const NAME = "…"`** in **`src/shared/utils/constants.ts`**, grouped under a **section comment** (same style as existing `// TOAST TYPES`, `// DATA TYPES`, …), and import via **`@shared/utils/constants`**.
- **Exception (allowed inline only):** user-visible copy that lives in **i18n** resources, or **test** files. Everything else goes through **`constants.ts`**.
- If a constant is **truly** private to one feature and would pollute shared, it **may** live in **`src/features/<domain>/constants.ts`**—**still** no inline magic strings in components/hooks/utils; import from that file.

**Must not:** duplicate the same literal in multiple files; reintroduce literals that already exist in **`constants.ts`**.

---
> Source: [espressif/esp-rainmaker-home](https://github.com/espressif/esp-rainmaker-home) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-04 -->
