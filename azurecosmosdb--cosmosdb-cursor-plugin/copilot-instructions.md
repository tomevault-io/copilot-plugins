## cosmosdb-cursor-plugin

> Azure Cosmos DB indexing strategies: composite indexes, excluded paths, index types, and spatial indexes


# Indexing Strategies

Strategic indexing reduces query costs while minimizing write overhead. Default indexing often includes unused paths.

## Use composite indexes for ORDER BY

Queries with `ORDER BY` on multiple properties require composite indexes. Without them, queries fail or consume excessive RU.

```json
// ❌ Bad - ORDER BY on multiple fields without composite index fails
// "SELECT * FROM c ORDER BY c.createdAt DESC, c.priority ASC"
// Error: "The order by query does not have a corresponding composite index"

// ✅ Good - add composite index in indexing policy
{
  "indexingPolicy": {
    "compositeIndexes": [
      [
        { "path": "/createdAt", "order": "descending" },
        { "path": "/priority", "order": "ascending" }
      ]
    ]
  }
}
```

```csharp
// Define in code
var indexingPolicy = new IndexingPolicy();
indexingPolicy.CompositeIndexes.Add(new Collection<CompositePath>
{
    new CompositePath { Path = "/createdAt", Order = CompositePathSortOrder.Descending },
    new CompositePath { Path = "/priority", Order = CompositePathSortOrder.Ascending }
});
```

## Exclude unused index paths

By default, Cosmos DB indexes all paths. Exclude large or never-queried fields to reduce write RU and storage.

```json
// ❌ Bad - indexing everything including large text fields
{
  "indexingPolicy": {
    "includedPaths": [{ "path": "/*" }]
  }
}

// ✅ Good - exclude fields never used in queries
{
  "indexingPolicy": {
    "includedPaths": [{ "path": "/*" }],
    "excludedPaths": [
      { "path": "/largeDescription/*" },
      { "path": "/internalMetadata/*" },
      { "path": "/\"_etag\"/?" }
    ]
  }
}
```

Exclude when:
- Fields are large text/JSON blobs never queried
- Fields are write-heavy but never filtered/sorted
- Vector embedding fields (always exclude from regular indexing)

## Choose appropriate index types

- **Range indexes** (default): Support equality, range, and ORDER BY queries
- **Spatial indexes**: Required for geo queries (`ST_DISTANCE`, `ST_WITHIN`)

```json
{
  "indexingPolicy": {
    "spatialIndexes": [
      {
        "path": "/location/*",
        "types": ["Point", "Polygon"]
      }
    ]
  }
}
```

## Understand indexing modes

- **Consistent** (default): Index updated synchronously with writes. Use for production.
- **None**: No indexing. Use only for pure key-value stores with point reads only.

```csharp
// ✅ Good - consistent indexing (default)
var policy = new IndexingPolicy { IndexingMode = IndexingMode.Consistent };

// Only for pure key-value lookups (no queries at all)
var policy = new IndexingPolicy { IndexingMode = IndexingMode.None };
```

## Add spatial indexes for geo queries

Required for geospatial queries using `ST_DISTANCE`, `ST_WITHIN`, `ST_INTERSECTS`.

```csharp
// Enable spatial indexing
indexingPolicy.SpatialIndexes.Add(new SpatialPath
{
    Path = "/location/*",
    SpatialTypes = new Collection<SpatialType> { SpatialType.Point, SpatialType.Polygon }
});

// Query nearby locations
var query = new QueryDefinition(
    "SELECT * FROM c WHERE ST_DISTANCE(c.location, @point) < @radiusMeters")
    .WithParameter("@point", new { type = "Point", coordinates = new[] { -122.12, 47.67 } })
    .WithParameter("@radiusMeters", 5000);
```

Reference: [Indexing policies](https://learn.microsoft.com/azure/cosmos-db/index-policy) | [Composite indexes](https://learn.microsoft.com/azure/cosmos-db/index-policy#composite-indexes)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/AzureCosmosDB)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/AzureCosmosDB)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
