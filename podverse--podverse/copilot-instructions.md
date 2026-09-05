## add-by-rss-data-contracts

> Keep Add-by-RSS parsed, persisted, and offline playback data contracts explicit across the server, parser mapping, and mobile SQLite


# Add-by-RSS data contracts

Add-by-RSS uses separate contracts by design. The server database stores followed-feed metadata; the
parse status carries a server payload; parser-mapping converts it into `AddByRSSMappedFeed`; and
mobile stores scalar list fields plus the mapped bundle for offline playback. Consistency means every
conversion is explicit, not that the schemas are identical.

When changing Add-by-RSS data, trace this complete path:

1. **Server:** `DTOAccountFollowingAddByRSSChannel`, the ORM entity, and
   `AddByRSSParseCacheEntry` define followed metadata and parse-status payloads. Parsed feed items are
   not stored in the server database.
2. **Parsed bundle:** `AddByRSSMappedFeed` is the canonical mapped result persisted in
   `add_by_rss_feed.mapped_feed_json`.
3. **Mobile record:** `MobileAddByRSSFeedRecord`, `AddByRssFeedRow`, `schema.ts`, and
   `migrations.ts` define scalar offline list fields and their SQLite representation.
4. **Playback:** `getMappedFeedByUrl`, `safeJsonParse`, and `toAddByRssPlaybackResourceData`
   reconstruct playback data from the bundle and per-account playback position.

Check identifiers, URLs, titles, images/enclosures, resource types, date units, nullability, and
stale-row behavior at every boundary. Update row mapping and upsert paths together. Cover fresh
parses, followed-list merges with an existing bundle, and missing or malformed offline payloads.
Use a forward-only SQLite migration for scalar column changes. Do not store Add-by-RSS items in
`channel_item`; that table is the bounded cache for directory-channel items.

Related: [`dto-changes-are-device-data-migrations`](/.cursor/rules/dto-changes-are-device-data-migrations.mdc)
— persisted DTOs remain device data migrations even when the SQLite shape differs from the server.

---
> Source: [podverse/podverse](https://github.com/podverse/podverse) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-05 -->
