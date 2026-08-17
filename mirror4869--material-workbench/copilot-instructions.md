## material-workbench

> Material Workbench is a Windows local-first media catalogue. Source code and

# AGENTS.md

## Project scope

Material Workbench is a Windows local-first media catalogue. Source code and
static assets live in this repository; real media, user metadata, generated
indexes, thumbnails, journals, logs, and local configuration do not.

## Safety rules

1. Never use a real media library in tests. Use temporary directories and
   synthetic fixtures.
2. Never commit or directly edit `user_data.json` or runtime journals.
3. Existing library files must not be silently moved, renamed, overwritten, or
   deleted.
4. Desktop intake defaults to a no-overwrite library copy. Moving the original
   is an explicit warned choice; a copy does not silently relink an external
   project or make its Desktop source safe to delete.
5. Destructive or transactional physical changes require preview, explicit
   confirmation, authoritative server-side revalidation, persistent recovery
   evidence, and rollback. First-ingest copy instead keeps the source, publishes
   atomically without overwrite, and removes only its own failed target.
6. Destructive and Adobe write APIs accept opaque IDs, not browser-provided
   filesystem paths.
7. Do not weaken reparse-point, hard-link, stale-plan, project-identity, or
   concurrent-write guards to make a test pass.
8. Adobe integration is optional and must fail closed when the external bridge
   or same-project runtime guard is unavailable.
9. Keep all text files UTF-8 and preserve stable asset ID algorithms.
10. Core workflows must remain usable without AI. Agent output is advisory and
    cannot bypass user confirmation, server-side validation, or recovery gates.
11. Browse and Organize are presentations of one library. New item actions and
    query concepts should share a command/query contract instead of creating a
    capability that exists in only one presentation.
12. Premiere, Blender, DaVinci Resolve, CapCut/Jianying, and other creative
    applications are optional capability-declared connectors. Keep reveal and
    copy-path usable without a connector, and never simulate a host write with
    UI automation or expose an action whose runtime guard is unavailable.
13. Keep `assets/js/workbench-query.js` pure and explicitly driven by item,
    query, and context inputs. The browser uses catalogue → query → bootstrap →
    oracle → main order; Organize/Material is the only migrated query adapter.
    Keep Browse/Home on its legacy implementation until independent equivalence
    tests pass, and do not imply that both interfaces already share all semantics.
14. `/api/bootstrap/status` must remain read-only: it may diagnose but must not
    scan, repair, install, or write. `assets/js/workbench-bootstrap.js` may
    synthesize an empty payload only for a source explicitly reported as
    `missing`; `invalid` or `unreadable` data blocks loading. Optional dependency
    failures must gate only the capabilities that require them, and every gated
    action must also fail closed inside its function body.
15. Keep `inspiration_cards_core.py` independent of the HTTP server. Inspiration
    registry v1 reads are zero-write; an explicit first v2 write requires its
    dedicated verified backup and one atomic save with one overall revision
    increment. V2 cards allow 0 to 6 ordered references, at most one cover in
    the first position, server-derived labels for local IDs, and URLs that the
    server records without fetching. Migration must inspect every local
    media/document reference in both schema versions, and exact deduplication
    must inspect every local asset reference.
16. The visible inspiration editor supports 0 to 6 ordered references. Keep v1
    reads and text-only edits zero-upgrade; send an explicit v2 request only
    after a real add, remove, reorder, or cover change. Local media and documents
    must come from the catalogue picker, never from a browser-provided ID or
    path. If the backend does not explicitly advertise `multi_ref_v2`, every
    rich-reference mutation and v2 save must fail closed while compatible v1
    text editing remains available.
17. Ordinary Recycle Bin deletion must check every matching v1/v2 local card
    reference and confirm it against the current inspiration-registry revision.
    Keep that confirmation independent from Adobe dependency confirmation; a
    drift in either revision performs no file move or metadata write. After a
    confirmed deletion, preserve the card and stored reference and resolve the
    source as missing.
18. Keep Daily Oracle local, optional, and non-authoritative. It may open only
    after an explicit user click and must never be restored as an open dialog by
    session recovery. The selected deck and normally browser-random salt belong
    only in `localStorage`; a fixed non-identifying fallback is allowed when
    random generation or storage is unavailable. None of these values may enter
    `user_data.json`, an index, or a server API. A draw stays stable for the
    user's local calendar day. Present the
    bundled traditional and original decks as entertainment and self-reflection,
    never as medical, legal, financial, safety, or other professional advice.
    Traditional content may use public-domain Major Arcana symbolism, but must
    not copy a modern deck's art, text, brand, or interactive assets.

## Validation

Run Python unittests, Node bridge tests, JavaScript syntax checks, and
`git diff --check`. Explain any omitted check in the pull request.

---
> Source: [mirror4869/material-workbench](https://github.com/mirror4869/material-workbench) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-17 -->
