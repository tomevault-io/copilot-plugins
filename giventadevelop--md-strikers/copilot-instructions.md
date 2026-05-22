## strapi-5-api-patterns

> Strapi 5 REST API patterns for published state, populate parameters, and list-endpoint fallbacks


# Strapi 5 REST API Patterns

## **Overview**

This rule defines correct patterns for Strapi 5 Content API calls, particularly for published-state filtering and populate parameters. Incorrect syntax causes 400 Bad Request errors.

## **Problem Solved**

- **status=published** – Can cause validation errors in Strapi 5 setups; use `filters[publishedAt][$notNull]=true` instead
- **Comma-separated populate** – `populate=cover,category,author` triggers 400s; Strapi 5 expects array-style populate
- **Category slug case mismatch** – Strapi may store slugs as `Main-News` while the filter uses `main-news`; use `$eqi` for case-insensitive match

## **Tenant Filtering (Required — No Fallback)**

All directory and news list/detail requests **must** use the tenant filter. Display only data scoped to the current tenant.

### **DO: Always filter by tenant**

- **Directory** (e.g. bishops, parishes): `filters[tenant][tenantId][$eq]=${tenantId}` — no fallback. If the API returns 200 with 0 items, show 0 items.
- **News** (articles, flash-news-items, advertisement-slots): Same. Always include `filters[tenant][tenantId][$eq]=${tenantId}`. Do **not** retry without the tenant filter when the response is 200 with empty data.

### **DON'T: Fall back to no-tenant when response is 200 with 0 items**

```typescript
// ❌ DON'T: Retry without tenant when tenant filter returns 0 items
if (list.length === 0) {
  const noTenant = await fetch('/articles?filters[publishedAt][$notNull]=true&...');
  list = noTenant.data; // Wrong — shows other tenants' data
}
```

When the API returns **200** with an empty list, the correct behavior is to show no items (and ensure entries in Strapi have the tenant relation set). The only exception is the **400 Bad Request** fallback below (different endpoints, different reason).

## **Published State Filter**

### **DO: Use filters[publishedAt][$notNull]=true**

```typescript
// ✅ DO: Filter for published documents
const query = `filters[publishedAt][$notNull]=true`;
```

### **DON'T: Use status=published**

```typescript
// ❌ DON'T: Can cause validation errors in Strapi 5
const query = `status=published`;
```

## **Populate Parameter (Array-Style)**

### **DO: Use array-style populate**

```typescript
// ✅ DO: Array-style populate for Strapi 5
const POPULATE = 'populate[0]=cover&populate[1]=category&populate[2]=author';

// Full example
const path = `/articles?filters[tenant][tenantId][$eq]=${tenantId}&filters[publishedAt][$notNull]=true&populate[0]=cover&populate[1]=category&populate[2]=author&sort=publishedAt:desc&pagination[limit]=10`;
```

### **DON'T: Use comma-separated populate**

```typescript
// ❌ DON'T: Triggers 400 Bad Request in Strapi 5
const POPULATE = 'populate=cover,category,author';
```

## **Example Article Query URLs**

### **Featured News**

```
GET /api/articles?filters[isFeatured][$eq]=true&filters[tenant][tenantId][$eq]=tenant_demo_002&filters[publishedAt][$notNull]=true&populate[0]=cover&populate[1]=category&populate[2]=author&sort=publishedAt:desc&pagination[limit]=6
```

### **Featured News** (use category slug like Main News, not isFeatured)

```
GET /api/articles?filters[category][slug][$eqi]=featured-news&filters[tenant][tenantId][$eq]=tenant_demo_002&filters[publishedAt][$notNull]=true&populate[0]=cover&populate[1]=category&populate[2]=author&sort=publishedAt:desc&pagination[limit]=6
```

If you have a "Featured News" category in Strapi (slug `Featured-News`), filter by category slug. The `isFeatured` boolean is an alternative; use category when articles are assigned to the Featured News category.

### **Main News** (use `$eqi` for case-insensitive slug)

```
GET /api/articles?filters[category][slug][$eqi]=main-news&filters[tenant][tenantId][$eq]=tenant_demo_002&filters[publishedAt][$notNull]=true&populate[0]=cover&populate[1]=category&populate[2]=author&sort=publishedAt:desc&pagination[limit]=10
```

### **Press Release / Most Read**

Same pattern: use `filters[publishedAt][$notNull]=true` and `populate[0]=...&populate[1]=...&populate[2]=...`

## **Category Slug Filter (Case-Insensitive)**

### **DO: Use $eqi for case-insensitive slug match**

Strapi may store category slugs with different casing (e.g. `Main-News`, `Press-Release`) from the UI. Slug filters are case-sensitive; use `$eqi` to match regardless of case:

```typescript
// ✅ DO: Case-insensitive category slug filter
'filters[category][slug][$eqi]=main-news'
'filters[category][slug][$eqi]=press-release'
```

### **DON'T: Use $eq for slugs (case-sensitive)**

```typescript
// ❌ DON'T: May not match "Main-News" when stored with different casing
'filters[category][slug][$eq]=main-news'
```

## **Article Detail Page (slug or id)**

The detail route `/mosc/news/[slug]` and `/syro/news/[slug]` accept slug, numeric id, or documentId:

1. **documentId** (Strapi 5, first when param looks like documentId): `filters[documentId][$eq]=<documentId>` — alphanumeric 15–35 chars (e.g. `o42fs0slvzzj9os0g9xtc11d`). Try documentId first when param matches `/^[a-z0-9]{15,35}$/` so URLs from Strapi admin or links using documentId work even when slug differs.
2. **Slug** (text): `filters[slug][$eqi]=<slug>` — preferred for SEO
3. **Numeric id** (e.g. `7`): `filters[id][$eq]=7` — when slug is empty

Article links use: `article.slug || article.documentId || article.id`. No Strapi model changes needed; ensure articles have slug populated for SEO-friendly URLs.

### **Article not found: Troubleshooting**

When the API returns 200 but 0 items and the frontend shows "Article not found":

- **Try documentId first**: Ensure `getArticleBySlug` tries `documentId` first when the URL param looks like a Strapi 5 documentId (alphanumeric, 15–35 chars). Admin URLs and many links use documentId; slug may differ.
- **Tenant**: Article must have `tenant.tenantId` matching `NEXT_PUBLIC_TENANT_ID` (e.g. `tenant_demo_002`). Check the tenant relation in Strapi admin. We do **not** retry without tenant — only tenant-scoped articles are shown.
- **publishedAt**: Article must be Published (not Draft). Filter `filters[publishedAt][$notNull]=true` excludes drafts. If the article is draft, set Status to Published in Strapi.

## **Advertisement Slots (Position: sidebar + top)**

Fetch both sidebar and top ads with `$or`:

```
filters[$or][0][position][$eq]=sidebar&filters[$or][1][position][$eq]=top
```

Split results client-side by `position` for sidebar vs top banner display.

## **Use Native Strapi 5 Response Format**

Do **not** send `Strapi-Response-Format: v4`. Use native Strapi 5 responses (flattened structure). The extraction helpers in `getNewsHomePageData.ts` (`getMediaUrl`, `getMediaAlt`, `attrs = raw?.attributes ?? raw`) support both Strapi 4 and Strapi 5, but the server runs Strapi 5 and we consume its native format. Implementation: [`src/lib/strapi.ts`](mdc:src/lib/strapi.ts) - `getStrapiHeaders()` does not include a v4 compatibility header.

## **Media/Relation URL Extraction (Strapi 4 vs 5)**

Strapi 5 uses a flattened response format; media URLs are no longer at `relation.data.attributes.url`.

### **DO: Use a helper that supports both formats**

```typescript
function getMediaUrl(media: unknown, baseUrl: string): string | null {
  if (!media || !baseUrl) return null;
  const m = media as Record<string, unknown>;
  let url: string | undefined;
  // Strapi 5: media.url or media.data.url
  url = m?.url as string | undefined;
  if (!url) url = (m?.data as Record<string, unknown>)?.url as string | undefined;
  if (!url) {
    // Strapi 4: media.data.attributes.url
    const attrs = (m?.data as Record<string, unknown>)?.attributes as Record<string, unknown> | undefined;
    url = attrs?.url as string | undefined;
  }
  if (!url) return null;
  return url.startsWith('http') ? url : `${baseUrl}${url}`;
}
```

### **DON'T: Assume only Strapi 4 structure**

```typescript
// ❌ DON'T: Only handles Strapi 4 - fails with Strapi 5 flattened format
const url = media?.data?.attributes?.url;
```

### **Apply to**

- Article `cover` → `coverUrl`
- Advertisement Slot `media` → `mediaUrl`
- Sidebar Promotional Block `thumbnail` → `thumbnailUrl`

## **Exception: Simple populate**

For single-field populate (e.g. `populate=media`), the simple format may work. Advertisement slots use `populate=media` successfully. If you encounter 400s with simple populate, switch to array-style: `populate[0]=media`.

## **List Endpoints: Pagination and 400-Only Fallback**

Use `pagination[page]=1` and `pagination[pageSize]=N` for list endpoints (not `pagination[limit]=N`). Some Strapi 5 list endpoints (e.g. `catholicos-entries`) return **400 Bad Request** when using certain query shapes; a **400-only** fallback is allowed in those cases.

### **DO: Use pagination[pageSize] for list endpoints**

```typescript
// ✅ DO: Strapi 5 list pagination
'pagination[pageSize]=1'
// or
'pagination[page]=1&pagination[pageSize]=1'
```

### **DON'T: Rely only on pagination[limit]**

```typescript
// ❌ DON'T: May trigger 400 on some Strapi 5 list endpoints
'pagination[limit]=1'
```

### **400-only fallback (not for 200 with 0 items)**

The two-step fallback (try without tenant, then with tenant) applies **only when the request returns 400 Bad Request** (syntax/validation error). It does **not** apply when the API returns **200** with 0 items — in that case, show 0 items and do not retry without tenant.

- **Articles, bishops, flash-news-items, advertisement-slots**: Always use tenant filter. Never retry without tenant when the response is 200 with empty data.
- **Exception**: Endpoints that return 400 with tenant + pagination (e.g. `catholicos-entries`) may use a 400-only fallback: try without tenant first; if that returns 200, use the result (e.g. for Catholicos image fallback in bishop detail).

```typescript
// ✅ DO: 400-only fallback — try without tenant only when first request returns 400
const pathsToTry = [
  `catholicos-entries?pagination[pageSize]=1&populate[0]=image`,
  `catholicos-entries?filters[tenant][tenantId][$eq]=${encodeURIComponent(tenantId)}&pagination[pageSize]=1&populate[0]=image`,
];
let res: Response | null = null;
for (const path of pathsToTry) {
  res = await fetch(`${base}/${path}`, { headers: getStrapiHeaders(), cache: 'no-store' });
  if (res.ok) break; // Use first successful response (200)
}
if (res?.ok) {
  const json = await res.json();
  // use json.data (array or single object)
}
```

### **Apply to**

- Directory **catholicos-entries** (Catholicos image fallback when bishops API returns no image) — 400-only fallback allowed
- **Not** for articles, bishops list, flash-news-items, advertisement-slots — those must use tenant filter only, no fallback when 200 with 0 items

### **Reference**

- Implementation: [`src/app/mosc/directory/bishops/getBishopsData.ts`](mdc:src/app/mosc/directory/bishops/getBishopsData.ts) – Catholicos image fallback with two-step request

## **Strapi Image Upload Recommendations (News Article Covers)**

Full specs: [`documentation/news_portal/strapi/image_upload_spec.md`](documentation/news_portal/strapi/image_upload_spec.md).

### **Suggested dimensions**

| Use | Size | Aspect |
|-----|------|--------|
| **Primary** | **1200 × 800 px** | 3:2 |
| Alternative (list) | 1200 × 900 px | 4:3 |
| Alternative (hero) | 1200 × 675 px | 16:9 |

### **Format and quality**

- **Format:** JPEG (or WebP if available)
- **Quality:** 80–85%
- **Target file size:** Under 200 KB per image

### **Rationale**

- **1200px** on the long side: enough for retina at max display (896px)
- **3:2 (1200×800):** works for both list cards and hero without heavy cropping
- **JPEG 80–85%:** balances quality and load time
- **Under 200 KB:** keeps page load fast on mobile

### **Composition**

- Put faces and important elements in the upper half (headers are now visible)
- Avoid important content at the very edges
- Use simple, neutral backgrounds where possible

## **References**

- Implementation: [`src/app/mosc/news/getNewsHomePageData.ts`](mdc:src/app/mosc/news/getNewsHomePageData.ts)
- Strapi client: [`src/lib/strapi.ts`](mdc:src/lib/strapi.ts)
- API reference: [`documentation/news_portal/strapi/api_reference.md`](mdc:documentation/news_portal/strapi/api_reference.md)

---
> Source: [giventadevelop/md-strikers](https://github.com/giventadevelop/md-strikers) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
