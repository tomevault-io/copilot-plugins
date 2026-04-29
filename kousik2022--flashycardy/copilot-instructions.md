## flashycardy

> Data reads via Server Components; writes via Server Actions; validate all action inputs with Zod (typed, no FormData types)


# Data access: reads in Server Components; writes in Server Actions

- **Data retrieval**: Must be done via **Server Components** (or server-only modules called by them). Do not fetch DB data in client components.
- **DB mutations**: **Inserts/updates/deletes** must be done via **Server Actions** only (`"use server"`).
- **Validation**: All data passed to Server Actions must be **validated with Zod**.
- **Types**: Server Action inputs must have a **TypeScript type** derived from the Zod schema. **Do not use `FormData` as the type**.

## Examples

```ts
// ✅ GOOD: Server Action with Zod-validated, typed input
"use server";
import { z } from "zod";

const RenameDeckInput = z.object({ deckId: z.string().uuid(), name: z.string().min(1).max(80) });
type RenameDeckInput = z.infer<typeof RenameDeckInput>;

export async function renameDeck(raw: RenameDeckInput) {
  const input = RenameDeckInput.parse(raw);
  // ... perform db update here ...
}
```

```ts
// ❌ BAD: mutation outside a Server Action
export async function deleteDeck(deckId: string) {
  // ... db.delete(...) ...
}

// ❌ BAD: using FormData as the input type (still must parse/validate with Zod)
export async function createDeck(data: FormData) {}
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/kousik2022) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
