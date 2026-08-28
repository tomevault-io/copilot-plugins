## mongo-weaver

> `remeda` is a dependency of all three packages. **If remeda has a function for it, use

# mongo-weaver

## Use remeda, not native JS

`remeda` is a dependency of all three packages. **If remeda has a function for it, use
remeda.** A native call is a deliberate exception, not a default — reach for one only
when there is no remeda equivalent, or when the remeda form would be strictly longer and
less clear (a single `.trim()`, a template string).

The point is uniformity: one vocabulary across the codebase, non-mutating by default, and
predicates that narrow types instead of merely returning booleans.

| Instead of                        | Write                                                          |
| --------------------------------- | -------------------------------------------------------------- |
| `xs.length === 0`, `!xs.length`   | `isEmpty(xs)` — also strings and objects                       |
| `Object.keys(o).length === 0`     | `isEmpty(o)`                                                   |
| `s === ''`                        | `isEmpty(s)`                                                   |
| `Object.keys/values/entries(o)`   | `keys(o)` / `values(o)` / `entries(o)`                         |
| `xs.map(f)` / `.filter(p)`        | `map(xs, f)` / `filter(xs, p)`                                 |
| `xs.find(p)` / `.flatMap(f)`      | `find(xs, p)` / `flatMap(xs, f)`                               |
| `xs.includes(x)`                  | `isIncludedIn(x, xs)`                                          |
| `xs[0]` / `xs.at(-1)`             | `first(xs)` / `last(xs)`                                       |
| `[...xs].sort(cmp)`               | `sortBy(xs, fn)` / `sort(xs, cmp)`                             |
| `xs.reduce((n, x) => n + x.k, 0)` | `sumBy(xs, (x) => x.k)`                                        |
| `new Set(xs)` for dedupe          | `unique(xs)` / `uniqueBy(xs, fn)`                              |
| `typeof v === 'string'` etc.      | `isString(v)`, `isNumber(v)`, `isArray(v)`, `isPlainObject(v)` |
| `v !== null && v !== undefined`   | `isDefined(v)` / `isNullish(v)`                                |
| `a === b` on objects              | `isDeepEqual(a, b)`                                            |
| `!p(x)` as a predicate            | `isNot(p)`                                                     |

Chain with `pipe` rather than nesting calls — `pipe(xs, map(f), filter(p))` reads in the
order it runs; `filter(map(xs, f), p)` reads inside out.

Two rules that override the table:

- **`isEmpty` narrows, but not point-free.** `isEmpty(s)` on a string narrows to `''`, so
  `!isEmpty(s)` is a real guard. Passed as a bare reference — `filter(isNot(isEmpty))` —
  it resolves to the `string | undefined` overload, which remeda deprecates in favour of
  `isEmptyish`, and lint fails. Call it in a lambda instead.
- **Never mutate.** `sort`/`sortBy`/`reverse` return new arrays; the native ones mutate in
  place, which has bitten query results shared between a table and its rail.

---
> Source: [theajr/mongo-weaver](https://github.com/theajr/mongo-weaver) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-28 -->
