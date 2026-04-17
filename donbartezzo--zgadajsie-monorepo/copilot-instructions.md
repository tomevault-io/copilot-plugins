## zgadajsie-monorepo

> Reguły frontendowe — aktywne przy pracy z plikami frontend/


## Obowiązkowe guide'y

Przy implementacji frontendowej ZAWSZE przeczytaj:

- `docs/styleguide-common.md`
- `docs/styleguide-frontend.md`

## ZABRONIONE w szablonach i komponentach

- domyślne kolory Tailwind: `gray-*`, `blue-*`, `slate-*`, `red-*`, `zinc-*`
- arbitralne hexy: `bg-[#abc]`
- prefixy `dark:`
- bezpośrednie używanie CSS vars w szablonach i komponentowych SCSS (np. `rgb(var(--color-*))`)
- `@apply` z custom theme colors w komponentowych stylesheets (Tailwind v4 nie widzi custom colors z JS config)

Używaj WYŁĄCZNIE semantycznych klas Tailwind w HTML: `primary-*`, `neutral-*`, `success-*`, `warning-*`, `danger-*`, `info-*`.

## Kluczowe zasady Angular

- Signals do stanu, `computed()` do pochodnych, `effect()` tylko dla side effects
- `ChangeDetectionStrategy.OnPush` domyślnie
- `input()` / `output()` zamiast dekoratorów `@Input()` / `@Output()`
- `inject()` zamiast konstruktorowego DI
- Nowa składnia: `@if`, `@for` (z `track`), `@switch`
- Sprawdź `shared/utils/index.ts` przed pisaniem nowej logiki

## Tailwind v4

- Global `styles.scss`: `@apply` działa (Tailwind załadowany)
- Component SCSS: NIE używaj `@apply` z custom theme — wstaw klasy Tailwind bezpośrednio w HTML
- Config: `tailwind.config.js` via `@config` directive
- Renamed utilities: `shadow-xs` (nie `shadow-sm`), `rounded-xs` (nie `rounded-sm`), `outline-hidden` (nie `outline-none`)

## Linting

- nie łącz `class="..."` z `[class]="..."`; buduj jedną wartość dla `[class]`
- nie neguj `async` pipe w szablonach
- `console.log` zakazany; dozwolone `console.warn` / `console.error`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/donbartezzo) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
