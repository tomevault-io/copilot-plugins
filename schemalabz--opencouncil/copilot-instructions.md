## auth

> This rules specifies how authentication checks are done on the OpenCouncil repo.


This rules specifies how authentication checks are done on the OpenCouncil repo.

1. Use the methods in src/lib/auth.ts.
2. Two main methods exist: isUserAuthorizedToEdit (returns a boolean) and withUserAuthorizedToEdit (throws if not authorized).
3. Both methods are asynchronous, and should be await-ed -- otherwise this can lead to serious authentication bugs.

Example usage:
    const editable = await isUserAuthorizedToEdit({ cityId: data.meeting.cityId });

or:
    await withUserAuthorizedToEdit({ partyId: params.partyId })

---
> Source: [schemalabz/opencouncil](https://github.com/schemalabz/opencouncil) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-03 -->
