## corevecdb

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build and Run Commands

```bash
# Build (release)
cargo build --release

# Run the server (uses config.toml)
cargo run --release

# Build only (compiles protobuf definitions via build.rs)
cargo build

# Test client (requires server running)
python test_client.py

# Run benchmark
python benchmark.py
```

## Architecture Overview

VectorDB is a Rust-based vector database with HNSW indexing, exposing both gRPC and HTTP APIs.

### Core Layers

1. **Storage** (`src/storage/mod.rs`)
   - `MemmapVectorStore`: Memory-mapped file for vector data (`data/vectors.bin`)
   - `QuantizedMemmapVectorStore`: Scalar quantized storage (75% memory reduction)
   - `SledMetadataStore`: Embedded key-value store for metadata (`data/meta.sled`)
   - `IndexedSledMetadataStore`: SledMetadataStore + PayloadIndex for fast filtering
   - Vectors are append-only; IDs are auto-assigned sequentially starting from 0

2. **Index** (`src/index/`)
   - `HnswIndexer`: Wraps `hnsw_rs` crate for approximate nearest neighbor search
   - `DynamicHnswIndex`: Deletion support with lazy repair and auto-rebuild
   - Uses L2 (Euclidean) distance; for cosine similarity, normalize vectors before insertion
   - Supports filtered search via `RoaringBitmap`
   - Index persists to `data/index.hnsw.*` files on shutdown

3. **Quantization** (`src/quantization/`)
   - `ScalarQuantizer`: float32 → uint8 compression (4x memory reduction)
   - `ProductQuantizer`: Subvector k-means compression (64x memory reduction)
   - Online training: learns parameters incrementally
   - ADC (Asymmetric Distance Computation): query in float32, candidates quantized

4. **Payload Index** (`src/payload/mod.rs`)
   - `PayloadIndex`: Inverted index for metadata filtering
   - `FieldIndex`: Per-field string index using `RoaringBitmap` per unique value
   - `NumericFieldIndex`: BTreeMap-based index for range queries (O(log n))
   - `FilterQuery` DSL: `Eq`, `In`, `Gt`, `Gte`, `Lt`, `Lte`, `Range`, `And`, `Or`, `Not`
   - Pre-filtering in HNSW search for indexed fields
   - Automatic fallback to post-filtering for non-indexed fields

5. **Collection** (`src/collection/mod.rs`)
   - `Collection`: Independent vector space with its own storage, index, and metadata
   - `CollectionManager`: Manages multiple collections (create, delete, list)
   - `CollectionConfig`: Per-collection settings (dim, distance, quantization, indexed_fields)
   - Each collection stored in `data/<collection_name>/` directory

6. **Text Index** (`src/text/mod.rs`)
   - `TextIndex`: BM25 full-text search index
   - Inverted index with term frequency tracking
   - IDF (Inverse Document Frequency) scoring
   - Hybrid search combining vector + text scores
   - Fusion methods: Weighted combination, RRF (Reciprocal Rank Fusion)

7. **WAL (Write-Ahead Log)** (`src/wal/mod.rs`)
   - `WriteAheadLog`: Crash recovery through durable operation logging
   - Operations logged before applying to index/storage
   - Checkpoint system for incremental saves and WAL truncation
   - CRC32 checksums for data integrity

8. **SIMD** (`src/simd/mod.rs`)
   - Hardware-accelerated distance functions (L2, Dot Product, Cosine)
   - Runtime CPU feature detection (AVX2, SSE4.1, NEON)
   - 4-8x speedup for vector operations
   - Optimized uint8 quantized vector distance

9. **Sharding** (`src/sharding/mod.rs`)
   - Distributed horizontal data partitioning
   - Consistent hashing with virtual nodes for even distribution
   - Cross-shard search with parallel result merging
   - Global ID encoding (shard ID + local ID)
   - Automatic shard routing by vector ID

10. **API** (`src/api/`)
    - `mod.rs`: gRPC service implementation (`VectorServiceImpl`)
    - `http.rs`: Axum HTTP handlers with multi-collection support
    - Legacy endpoints use "default" collection for backward compatibility

11. **Proto** (`src/proto/vectordb.proto`)
   - Defines `VectorService` with `Upsert`, `Search`, `Get`, `HybridSearch`, `TextSearch` RPCs
   - Compiled via `tonic-build` in `build.rs`

### Request Flow

```
Client → HTTP (port 3000) or gRPC (port 50051)
      → VectorServiceImpl
      → MemmapVectorStore (vector data)
      → HnswIndexer (similarity search)
      → SledMetadataStore (metadata filtering)
```

### Configuration

`config.toml` controls:
- `server.grpc_port`, `server.http_port`, `server.data_dir`
- `index.dim` (vector dimension), `index.max_elements`, `index.m`, `index.ef_construction`
- `quantization.enabled`, `quantization.keep_originals`, `quantization.rerank_oversample`
- `payload.index_enabled`, `payload.indexed_fields`, `payload.numeric_fields`
- `cache.query_cache_enabled`, `cache.query_cache_size`, `cache.query_cache_ttl_seconds`
- `cache.filter_cache_enabled`, `cache.filter_cache_size`

Environment override: `APP_SERVER__GRPC_PORT=50052` etc.

## HTTP API Endpoints

### Collection Management
- `GET /collections` - List all collections
- `POST /collections` - Create new collection
- `GET /collections/:name` - Get collection info
- `DELETE /collections/:name` - Delete collection

### Vector Operations (with collection)
- `POST /collections/:name/upsert` - Insert single vector
- `POST /collections/:name/upsert_batch` - Batch insert vectors (~10x faster)
- `POST /collections/:name/search` - Search vectors
- `GET /collections/:name/vectors/:id` - Get vector by ID
- `PUT /collections/:name/vectors/:id` - Update vector (delete + insert, new ID)
- `PATCH /collections/:name/vectors/:id` - Update metadata only (same ID)
- `DELETE /collections/:name/vectors/:id` - Delete vector (soft delete)
- `POST /collections/:name/delete_batch` - Batch delete vectors

### Snapshot Operations
- `GET /collections/:name/snapshots` - List snapshots
- `POST /collections/:name/snapshots` - Create snapshot
- `POST /snapshots/:snapshot_name/restore` - Restore from snapshot
- `DELETE /snapshots/:snapshot_name` - Delete snapshot

### WAL & Maintenance Operations
- `POST /collections/:name/checkpoint` - Create WAL checkpoint
- `GET /collections/:name/wal_stats` - Get WAL statistics
- `POST /collections/:name/compact` - Compact collection (remove deleted vectors)
- `POST /checkpoint_all` - Checkpoint all collections

### Streaming Operations
- `POST /collections/:name/stream_upsert` - SSE streaming bulk insert
- `POST /collections/:name/stream_search` - SSE streaming search for large k values

### Hybrid Search (Vector + Text)
- `POST /collections/:name/hybrid_search` - Combined vector + BM25 text search
- `POST /collections/:name/text_search` - Text-only BM25 search

### Multi-Vector Search
- `POST /collections/:name/multi_search` - Search with multiple query vectors

### Sparse Vector Operations
- `POST /collections/:name/sparse_upsert` - Insert sparse vector
- `POST /collections/:name/sparse_upsert_batch` - Batch insert sparse vectors
- `POST /collections/:name/sparse_search` - Search sparse vectors
- `POST /collections/:name/hybrid_dense_sparse` - Hybrid dense + sparse search

### Versioning Operations
- `POST /collections/:name/versioned/upsert` - Insert/update with version tracking
- `GET /collections/:name/versioned/vectors/:id` - Get latest version
- `GET /collections/:name/versioned/vectors/:id/history` - Get version history
- `POST /collections/:name/versioned/vectors/:id/at` - Get version at timestamp
- `POST /collections/:name/versioned/vectors/:id/rollback` - Rollback to version
- `POST /collections/:name/versioned/snapshot` - Get snapshot at timestamp
- `GET /collections/:name/versioned/stats` - Get versioning statistics

### Cache & Index Stats
- `GET /collections/:name/cache_stats` - Get query and filter cache statistics
- `GET /collections/:name/index_stats` - Get adaptive index statistics

### System Info
- `GET /simd` - Get SIMD capability information
- `GET /metrics` - Prometheus metrics

### Sharding Management
- `GET /sharding/config` - Get sharding status and configuration
- `POST /sharding/config` - Update sharding configuration
- `GET /sharding/shards` - List all shards
- `GET /sharding/shards/:shard_id` - Get shard info
- `POST /sharding/shards` - Add a new shard
- `DELETE /sharding/shards/:shard_id` - Remove a shard
- `GET /sharding/route/:vector_id` - Get shard for a vector ID

### Legacy Endpoints (uses "default" collection)
- `POST /upsert` - Insert vector with metadata
- `POST /upsert_batch` - Batch insert vectors
- `POST /search` - k-NN search with optional metadata/ID filtering
- `GET /vectors/:id` - Retrieve vector by ID
- `GET /stats` - Get vector count and status

Static files served from `ui/` directory at root

## Key Implementation Details

- Vector IDs are internal (0-indexed, auto-incremented); user-provided IDs in upsert are currently ignored
- Search supports two filtering modes: pre-filter via `filter_ids` or `filter_id_range` (RoaringBitmap) and post-filter via metadata `filter`
- Post-filtering uses Rayon for parallel metadata lookups
- Index is saved on graceful shutdown (SIGINT/SIGTERM)

## Performance Optimizations

### Storage Layer
- **Write Buffer**: Batches 256 vectors before flushing to disk (reduces fsync overhead)
- **parking_lot locks**: Uses `parking_lot::Mutex` and `RwLock` instead of std for better contention handling
- **Atomic counters**: Lock-free vector count tracking via `AtomicUsize`
- **Mmap optimization**: Tracks mmap length atomically, auto-refreshes after flush

### Metadata Layer
- **LRU Cache**: 10,000 entry cache for metadata lookups (32x speedup on cache hit)
- **Write-through policy**: Cache updated on insert

### ID Filtering
- **`filter_ids`**: Uses `from_sorted_iter` for O(n) bitmap creation
- **`filter_id_range`**: Uses `insert_range` for O(1) contiguous range filters

### Query Layer Caching (`src/cache/`)

#### Query Result Cache
LRU cache for search results with TTL-based expiration.

```rust
use vectordb::cache::{QueryCache, QueryCacheKey};

let cache = QueryCache::new(1000, Duration::from_secs(60));

// Cache key includes: collection, vector_hash, k, filter_hash
let key = QueryCacheKey::new("products", &query_vector, 10, &filter);

// Check cache
if let Some(results) = cache.get(&key) {
    return results;
}

// Store in cache
cache.put(key, results.clone());
```

Configuration:
```toml
[cache]
query_cache_enabled = true
query_cache_size = 1000      # Max cached queries
query_cache_ttl_seconds = 60 # TTL in seconds
```

#### Filter Bitmap Cache
Caches RoaringBitmap results for repeated filter conditions.

```rust
use vectordb::cache::{FilterBitmapCache, FilterCacheStats};

let cache = FilterBitmapCache::new(500);

// Cache filter results
let filter = vec![("category".into(), "electronics".into())];
cache.put(&filter, bitmap.clone());

// Retrieve cached bitmap
let bitmap = cache.get(&filter);

// Field-level invalidation on insert/update
cache.invalidate_field("category");
```

### Query Planning (`src/query/planner.rs`)

Estimates filter selectivity and optimizes filter ordering.

```rust
use vectordb::query::{QueryPlanner, FilterOrder, FilterStrategy};

let planner = QueryPlanner::new();

// Observe field values during insert
planner.observe_field("category", "electronics");
planner.observe_field("status", "active");

// Estimate selectivity (0.0 = very selective, 1.0 = no filtering)
let selectivity = planner.estimate_selectivity("category", "electronics");

// Optimize filter order (most selective first)
let filters = vec![
    FilterOrder::new("status", "active"),      // 90% of vectors
    FilterOrder::new("category", "electronics"), // 10% of vectors
];
let optimized = planner.optimize_filter_order(filters);
// Result: ["category", "status"] - electronics filter applied first

// Get recommended strategy
let plan = planner.plan_filters(filters);
match plan.strategy {
    FilterStrategy::PreFilter => // selectivity < 10%
    FilterStrategy::Hybrid => // 10-50%
    FilterStrategy::PostFilter => // > 50%
    FilterStrategy::NoFilter => // 100%
}
```

### Adaptive Index Selection (`src/index/adaptive.rs`)

Automatically switches between brute-force and HNSW based on collection size.

```rust
use vectordb::index::adaptive::{AdaptiveIndexer, AdaptiveIndexConfig, IndexType};

let config = AdaptiveIndexConfig {
    enabled: true,
    brute_force_threshold: 2000,  // Switch to HNSW at 2000 vectors
    ..Default::default()
};

let indexer = AdaptiveIndexer::new(config, vector_store, 128);

// Check current index type
match indexer.current_type() {
    IndexType::BruteForce => // SIMD-accelerated linear scan
    IndexType::Hnsw => // Approximate nearest neighbor
}

// Insert triggers automatic switching
let switched = indexer.insert(id, &vector)?;
if switched {
    println!("Switched to HNSW index");
}

// Search uses appropriate index
let results = indexer.search(&query, 10, filter)?;
```

| Collection Size | Index Type | Reason |
|-----------------|------------|--------|
| < 2000 | Brute-force | Lower overhead, exact results |
| >= 2000 | HNSW | Better scalability |

### Index Pre-warming (`src/index/prewarm.rs`)

Loads memory-mapped pages into OS cache for reduced cold-start latency.

```rust
use vectordb::index::prewarm::{prewarm_file, prewarm_collection, PrewarmConfig};

let config = PrewarmConfig {
    enabled: true,
    max_pages: 0,       // 0 = load all pages
    page_size: 4096,
    sequential: true,   // Sequential access pattern
};

// Pre-warm a single file
let stats = prewarm_file(Path::new("index.hnsw.graph"), &config)?;
println!("Loaded {} pages ({} bytes)", stats.pages_loaded, stats.bytes_loaded);

// Pre-warm entire collection
let stats = prewarm_collection(Path::new("data/products"), &config)?;
```

Files pre-warmed:
- `index.hnsw.graph` - HNSW graph structure
- `index.hnsw.data` - HNSW node data
- `vectors.bin` - Vector storage
- `vectors.quant` - Quantized vectors

### Upper Layer Cache (`src/index/layer_cache.rs`)

Caches HNSW entry points and region seeds for faster search initialization.

```rust
use vectordb::index::layer_cache::{HnswLayerCache, LayerCacheConfig};

let config = LayerCacheConfig {
    max_entry_points: 100,
    max_regions: 1000,
    region_hash_dims: 8,   // First 8 dimensions for hashing
    region_buckets: 10,
};

let cache = HnswLayerCache::new(config);

// Add global entry points
cache.add_entry_point(vector_id);

// Get starting points for a query
let starting_points = cache.get_starting_points(&query, 5);

// Record successful search to update cache
cache.record_success(&query, &results, entry_point_used);

// Check statistics
let stats = cache.stats();
println!("Hit rate: {:.1}%", stats.hit_rate());
```

### Tiered Storage (`src/storage/tiered.rs`)

Two-tier storage with hot (in-memory) and cold (mmap) tiers.

```rust
use vectordb::storage::tiered::{TieredVectorStore, TieredConfig};

let config = TieredConfig {
    enabled: true,
    hot_tier_max_vectors: 10_000,
    demotion_threshold_secs: 3600,  // 1 hour inactivity
    promotion_access_threshold: 3,
    promote_on_access: true,
};

let store = TieredVectorStore::new("data/vectors.bin", 128, config)?;

// Insert (starts in hot tier)
let id = store.insert(&vector)?;

// Get (promotes to hot if in cold tier)
let vector = store.get(id)?;

// Run maintenance (demote stale vectors)
store.maintain();

// Check statistics
let stats = store.stats();
println!("Hot tier: {} vectors ({:.1}% hit rate)",
         stats.hot_tier_vectors, stats.hot_hit_rate());
println!("Promotions: {}, Demotions: {}",
         stats.promotions, stats.demotions);
```

| Tier | Storage | Access Time | Capacity |
|------|---------|-------------|----------|
| Hot | In-memory HashMap | ~100ns | Configurable (default 10K) |
| Cold | Memory-mapped file | ~1μs | Unlimited |

### LZ4 Compression (`src/storage/compression.rs`)

Block-based LZ4 compression for vector segments.

```rust
use vectordb::storage::compression::{CompressedSegment, CompressionConfig};

let config = CompressionConfig {
    block_size: 256,  // Vectors per block
    enabled: true,
};

let mut segment = CompressedSegment::new(128, config);

// Add vectors
for vector in vectors {
    segment.add_vector(&vector)?;
}
segment.finalize();

// Random access (decompresses block on demand)
let vector = segment.get_vector(42)?;

// Check compression stats
let stats = segment.stats();
println!("Compression ratio: {:.1}%", stats.compression_ratio * 100.0);
println!("Space saved: {:.1}%", stats.space_saved_percent);

// Serialize/deserialize
let bytes = segment.to_bytes()?;
let restored = CompressedSegment::from_bytes(&bytes)?;
```

| Data Type | Typical Compression |
|-----------|---------------------|
| Random vectors | 60-80% of original |
| Similar vectors | 20-40% of original |
| Normalized vectors | 50-70% of original |

### Segmented Storage (`src/storage/segmented.rs`)

Fixed-size segments for better scalability and maintenance.

```rust
use vectordb::storage::segmented::{SegmentedVectorStore, SegmentConfig, SegmentState};

let config = SegmentConfig {
    segment_size: 100_000,  // Vectors per segment
    auto_seal: true,
    compress_sealed: false,
};

let store = SegmentedVectorStore::new("data/segments", 128, config)?;

// Insert (routes to active segment)
let global_id = store.insert(&vector)?;

// Get (routes to correct segment)
let vector = store.get(global_id)?;

// Check segment info
let stats = store.stats();
for info in &stats.segment_infos {
    println!("Segment {}: {:?}, {} vectors",
             info.id, info.state, info.vector_count);
}
```

**Global ID Encoding:**
```
global_id = segment_index * segment_size + local_id
```

| Segment State | Description |
|---------------|-------------|
| Active | Accepting new vectors |
| Sealed | Read-only, full capacity |
| Compressed | Sealed + LZ4 compressed |

## Scalar Quantization

Reduces memory usage by 75% by compressing float32 vectors to uint8.

### Configuration
```toml
[quantization]
enabled = true
keep_originals = false  # Set true for reranking with original vectors
rerank_oversample = 3   # Fetch 3x candidates for reranking
```

### How It Works
1. **Training**: Learns min/max per dimension online
2. **Encoding**: `uint8 = (float32 - min) / (max - min) * 255`
3. **Distance**: Asymmetric computation (query=float32, candidate=uint8)

### Memory Comparison
| Vectors | float32 | uint8 (quantized) | Savings |
|---------|---------|-------------------|---------|
| 10,000 | 5.12 MB | 1.28 MB | 75% |
| 100,000 | 51.2 MB | 12.8 MB | 75% |
| 1,000,000 | 512 MB | 128 MB | 75% |

## Product Quantization (PQ)

Achieves 90%+ memory reduction by dividing vectors into subvectors and quantizing each independently using k-means clustering.

### Architecture
```
Vector (128-dim, 512 bytes)
   │
   ▼ Split into M subvectors
┌────┬────┬────┬────┬────┬────┬────┬────┐
│ 16 │ 16 │ 16 │ 16 │ 16 │ 16 │ 16 │ 16 │  (M=8 subvectors)
└────┴────┴────┴────┴────┴────┴────┴────┘
   │
   ▼ K-means (K=256 centroids per subvector)
┌────┬────┬────┬────┬────┬────┬────┬────┐
│ u8 │ u8 │ u8 │ u8 │ u8 │ u8 │ u8 │ u8 │  PQ code (8 bytes)
└────┴────┴────┴────┴────┴────┴────┴────┘

Compression: 512 bytes → 8 bytes = 98.4% reduction
```

### Key Concepts
- **Codebook**: K centroids per subvector learned via k-means
- **PQ Code**: Vector of centroid indices (one per subvector)
- **ADC (Asymmetric Distance Computation)**: Query in float32, candidates in PQ codes

### Usage
```rust
use vectordb::quantization::pq::{ProductQuantizer, PQConfig, PQVectorStore};

// Create PQ for 128-dim vectors
let config = PQConfig::for_dim(128);
let mut pq = ProductQuantizer::new(config);

// Train on dataset
pq.train(&training_vectors)?;

// Encode vectors
let code = pq.encode(&vector)?;  // 8 bytes instead of 512

// Search with ADC (fast lookup-table based)
let results = pq.search_adc(&query, &all_codes, k);

// Or use PQVectorStore for complete storage solution
let store = PQVectorStore::new(128, true, 3);  // dim, keep_originals, rerank_factor
store.insert_batch(vectors)?;
let results = store.search(&query, 10)?;
```

### Memory Comparison (vs Scalar Quantization)
| Vectors (128-dim) | float32 | Scalar (75%) | PQ (98%) |
|-------------------|---------|--------------|----------|
| 10,000 | 5.12 MB | 1.28 MB | 80 KB |
| 100,000 | 51.2 MB | 12.8 MB | 800 KB |
| 1,000,000 | 512 MB | 128 MB | 8 MB |

### Accuracy Trade-offs
| Method | Compression | Recall@10 (typical) |
|--------|-------------|---------------------|
| float32 | 1x | 100% |
| Scalar (SQ) | 4x | 99%+ |
| PQ (M=8) | 64x | 90-95% |
| PQ + Reranking | 64x | 98%+ |

### Configuration Options
```rust
PQConfig {
    dim: 128,              // Vector dimension
    num_subvectors: 8,     // M - more = better accuracy, larger codes
    num_centroids: 256,    // K - typically 256 for u8 codes
    kmeans_iterations: 25, // Training iterations
}
```

## Payload Index

Inverted index for fast metadata filtering using Rust functional programming patterns.

### Configuration
```toml
[payload]
index_enabled = true
indexed_fields = ["category", "type", "status"]  # String fields
numeric_fields = ["price", "rating", "timestamp"] # Numeric fields (range queries)
```

### FilterQuery DSL

#### String Filters
```rust
// Equality
FilterQuery::eq("category", "electronics")

// In (multiple values)
FilterQuery::in_set("status", ["active", "pending"])
```

#### Range Filters (Numeric)
```rust
// Greater than: price > 100
FilterQuery::gt("price", 100)

// Greater than or equal: price >= 100
FilterQuery::gte("price", 100)

// Less than: price < 500
FilterQuery::lt("price", 500)

// Less than or equal: price <= 500
FilterQuery::lte("price", 500)

// Range (inclusive): 100 <= price <= 500
FilterQuery::range("price", 100, 500)

// Float variants (auto-converts to i64 with 6 decimal precision)
FilterQuery::gte_f("rating", 4.0)
FilterQuery::range_f("score", 0.5, 0.95)
```

#### Logical Operators
```rust
// AND: category=electronics AND price < 300
FilterQuery::and(vec![
    FilterQuery::eq("category", "electronics"),
    FilterQuery::lt("price", 300),
])

// OR: status=active OR status=pending
FilterQuery::or(vec![
    FilterQuery::eq("status", "active"),
    FilterQuery::eq("status", "pending"),
])

// NOT (requires universe bitmap)
FilterQuery::not(FilterQuery::eq("status", "deleted"))
```

### Search Filtering Flow
1. **Indexed fields**: Pre-filter with PayloadIndex → RoaringBitmap → HNSW search
2. **Non-indexed fields**: HNSW search → Post-filter with metadata lookup
3. **Combined**: Bitmap intersection for multiple filter types

### HTTP Range Filter API

Numeric range queries are available directly through the HTTP API.

#### Request Format
```json
{
  "vector": [...],
  "k": 10,
  "filter": {"category": "electronics"},
  "range_filters": [
    {"op": "gt", "field": "price", "value": 100},
    {"op": "lte", "field": "rating", "value": 5.0}
  ]
}
```

#### Supported Operators
| Operator | JSON Format | Description |
|----------|-------------|-------------|
| `gt` | `{"op": "gt", "field": "price", "value": 100}` | price > 100 |
| `gte` | `{"op": "gte", "field": "price", "value": 100}` | price >= 100 |
| `lt` | `{"op": "lt", "field": "price", "value": 100}` | price < 100 |
| `lte` | `{"op": "lte", "field": "price", "value": 100}` | price <= 100 |
| `range` | `{"op": "range", "field": "price", "min": 50, "max": 200}` | 50 <= price <= 200 |
| `between` | `{"op": "between", "field": "price", "min": 50, "max": 200}` | 50 < price < 200 |

#### Example Usage
```bash
# Find products with price > 100 AND rating >= 4.0
curl -X POST http://localhost:3000/collections/products/search \
  -H "Content-Type: application/json" \
  -d '{
    "vector": [...],
    "k": 10,
    "filter": {},
    "range_filters": [
      {"op": "gt", "field": "price", "value": 100},
      {"op": "gte", "field": "rating", "value": 4.0}
    ]
  }'

# Combine with string filter
curl -X POST http://localhost:3000/collections/products/search \
  -d '{
    "vector": [...],
    "k": 10,
    "filter": {"category": "electronics"},
    "range_filters": [
      {"op": "range", "field": "price", "min": 50, "max": 500}
    ]
  }'
```

### Performance
| Filter Type | Complexity | Notes |
|-------------|------------|-------|
| String field (Eq) | O(1) | HashMap lookup |
| String field (In) | O(k) | k = number of values |
| Numeric range (Gt/Lt/Range) | O(log n + m) | BTreeMap range scan, m = matches |
| Non-indexed field | O(N) | Post-filter scan |
| Combined filters | O(1) | Bitmap intersection |

### Functional Programming Patterns Used
- **Iterator chains**: `filter_map`, `fold`, `reduce`
- **Pattern matching**: FilterQuery enum dispatch
- **Closures**: Lazy evaluation in bitmap operations
- **Option/Result monads**: Safe error handling

## Multi-Collection

Each collection is an independent vector space with its own storage, index, and configuration.

### Create Collection
```bash
curl -X POST http://localhost:3000/collections \
  -H "Content-Type: application/json" \
  -d '{
    "name": "products",
    "dim": 128,
    "distance": "euclidean",
    "quantization_enabled": false,
    "indexed_fields": ["category", "brand"],
    "numeric_fields": ["price", "rating"]
  }'
```

### Collection Operations
```bash
# List all collections
curl http://localhost:3000/collections

# Get collection info
curl http://localhost:3000/collections/products

# Delete collection
curl -X DELETE http://localhost:3000/collections/products
```

### Vector Operations with Collection
```bash
# Upsert to specific collection
curl -X POST http://localhost:3000/collections/products/upsert \
  -d '{"vector": [...], "metadata": {"category": "electronics"}}'

# Search in specific collection
curl -X POST http://localhost:3000/collections/products/search \
  -d '{"vector": [...], "k": 10, "filter": {"category": "electronics"}}'

# Get vector from collection
curl http://localhost:3000/collections/products/vectors/0
```

### Directory Structure
```
data/
├── products/           # "products" collection
│   ├── config.json     # Collection configuration
│   ├── vectors.bin     # Vector storage
│   ├── index.hnsw.*    # HNSW index files
│   └── meta.sled/      # Metadata store
├── users/              # "users" collection
│   └── ...
└── default/            # Default collection (legacy endpoints)
    └── ...
```

### Collection Config Options
| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `name` | string | required | Collection name |
| `dim` | int | required | Vector dimension |
| `distance` | string | "euclidean" | Distance metric (euclidean, cosine) |
| `quantization_enabled` | bool | false | Enable scalar quantization |
| `indexed_fields` | string[] | [] | String fields for exact match filtering |
| `numeric_fields` | string[] | [] | Numeric fields for range queries |

## Update Vector

Update vectors or metadata. Two methods available:

### PUT - Full Vector Update
Replaces the vector and metadata. Since storage is append-only, this creates a new ID.

```bash
curl -X PUT http://localhost:3000/collections/products/vectors/123 \
  -H "Content-Type: application/json" \
  -d '{
    "vector": [0.1, 0.2, ...],
    "metadata": {"category": "electronics", "price": "299"}
  }'
```

Response:
```json
{
  "old_id": 123,
  "new_id": 456,
  "collection": "products",
  "updated": true
}
```

**Note**: The old ID is soft-deleted, and a new ID is assigned. Update your references to use the new ID.

### PATCH - Metadata Only Update
Updates metadata without changing the vector. Keeps the same ID.

```bash
curl -X PATCH http://localhost:3000/collections/products/vectors/123 \
  -H "Content-Type: application/json" \
  -d '{
    "metadata": {"price": "249", "on_sale": "true"}
  }'
```

Response:
```json
{
  "id": 123,
  "collection": "products",
  "updated": true
}
```

### When to Use Which
| Method | Use Case | ID Changes |
|--------|----------|------------|
| PUT | Change vector values | Yes (new ID) |
| PATCH | Change metadata only | No (same ID) |

### HTTP Status Codes
| Code | Meaning |
|------|---------|
| 200 | Successfully updated |
| 404 | Vector ID not found |
| 410 | Vector already deleted |

## Delete Vector

Soft delete vectors using tombstones. Deleted vectors are excluded from search results.

### API Endpoints
```bash
# Delete single vector
DELETE /collections/:name/vectors/:id

# Batch delete
POST /collections/:name/delete_batch
```

### Delete Single Vector
```bash
curl -X DELETE http://localhost:3000/collections/products/vectors/123
```

Response:
```json
{"id": 123, "collection": "products", "deleted": true}
```

### Batch Delete
```bash
curl -X POST http://localhost:3000/collections/products/delete_batch \
  -H "Content-Type: application/json" \
  -d '{"ids": [1, 2, 3, 4, 5]}'
```

Response:
```json
{"collection": "products", "deleted_count": 5, "requested_count": 5}
```

### HTTP Status Codes
| Code | Meaning |
|------|---------|
| 200 | Successfully deleted |
| 404 | Vector ID not found |
| 410 | Already deleted (Gone) |

### Collection Info with Delete Count
```json
{
  "name": "products",
  "dim": 128,
  "vector_count": 950,    // Active vectors
  "deleted_count": 50,    // Deleted vectors
  ...
}
```

### Implementation Details
- **Soft delete**: Uses RoaringBitmap to track deleted IDs
- **Persisted**: Deleted IDs saved to `deleted.bin` on flush
- **Search filter**: Deleted IDs automatically excluded from search results
- **Get returns 410**: Attempting to get deleted vector returns HTTP 410 Gone

### Storage
```
data/products/
├── vectors.bin       # Vector data (includes deleted)
├── deleted.bin       # Deleted ID bitmap
└── ...
```

## Compaction (Dynamic Deletion Recovery)

Compaction removes deleted vectors from storage and rebuilds the HNSW index for optimal performance.

### Why Compact?
- **Memory Recovery**: Deleted vectors still consume storage space
- **Search Quality**: HNSW graph traversal may visit deleted node locations
- **ID Space**: Reclaim IDs for new insertions

### API
```bash
# Compact a collection (removes deleted vectors, rebuilds index)
POST /collections/:name/compact
```

### Response
```json
{
  "vectors_before": 10000,
  "vectors_after": 8500,
  "vectors_removed": 1500,
  "bytes_reclaimed": 768000,
  "id_mapping": {"0": 0, "2": 1, "3": 2, ...}
}
```

### When to Compact
| Deletion Ratio | Recommendation |
|----------------|----------------|
| < 10% | No action needed |
| 10-30% | Optional, for memory recovery |
| > 30% | Recommended for search quality |
| > 50% | Required for optimal performance |

### Compaction Process
1. **Collect active vectors**: Skip deleted IDs
2. **Create new storage**: Write only active vectors
3. **Rebuild index**: Fresh HNSW graph without dead nodes
4. **Update metadata**: Migrate with new ID mapping
5. **Swap files**: Atomic replacement of old files

### ID Remapping
After compaction, vector IDs change. The response includes `id_mapping` for updating references:
```
Old ID 0 → New ID 0
Old ID 2 → New ID 1  (ID 1 was deleted)
Old ID 3 → New ID 2
...
```

### Dynamic Index Module (`src/index/dynamic.rs`)
For applications requiring more granular control:

```rust
use vectordb::index::dynamic::{DynamicHnswIndex, DynamicIndexConfig};

let config = DynamicIndexConfig::new(128, 100_000)
    .with_metric(DistanceMetric::Euclidean);

let index = DynamicHnswIndex::new(config);

// Insert
index.insert(0, &vector)?;

// Delete (immediate soft delete)
index.delete(1);

// Search (automatically excludes deleted)
let results = index.search(&query, 10)?;

// Check if rebuild needed (deletion_ratio > 30%)
if index.needs_rebuild() {
    let new_index = index.rebuild()?;
}

// Get stats
let stats = index.stats();
println!("Deletion ratio: {:.1}%", stats.deletion_ratio * 100.0);
```

## Snapshot/Backup

Create and restore backups of collections.

### API Endpoints
```bash
# List snapshots for a collection
GET /collections/:name/snapshots

# Create a snapshot
POST /collections/:name/snapshots

# Restore from snapshot
POST /snapshots/:snapshot_name/restore

# Delete a snapshot
DELETE /snapshots/:snapshot_name
```

### Create Snapshot
```bash
curl -X POST http://localhost:3000/collections/products/snapshots
```

Response:
```json
{
  "name": "products_20240320_143052",
  "collection": "products",
  "created_at": "2024-03-20T14:30:52Z",
  "vector_count": 10000,
  "size_bytes": 5242880
}
```

### List Snapshots
```bash
curl http://localhost:3000/collections/products/snapshots
```

### Restore from Snapshot
```bash
# Restore with original name (collection must not exist)
curl -X POST http://localhost:3000/snapshots/products_20240320_143052/restore \
  -H "Content-Type: application/json" \
  -d '{}'

# Restore with new name
curl -X POST http://localhost:3000/snapshots/products_20240320_143052/restore \
  -d '{"new_name": "products_restored"}'
```

### Delete Snapshot
```bash
curl -X DELETE http://localhost:3000/snapshots/products_20240320_143052
```

### Storage Structure
```
data/
├── products/           # Active collection
│   └── ...
└── _snapshots/         # Snapshot storage
    ├── products_20240320_143052/
    │   ├── snapshot.json   # Metadata
    │   ├── config.json
    │   ├── vectors.bin
    │   ├── index.hnsw.*
    │   └── meta.sled/
    └── products_20240319_120000/
        └── ...
```

### Use Cases
- **Backup before risky operations**: Create snapshot before bulk deletes
- **Point-in-time recovery**: Restore to previous state
- **Clone collections**: Restore with new name for testing
- **Migration**: Move collections between environments

## Write-Ahead Log (WAL)

Crash recovery mechanism using durable operation logging. Operations are written to WAL before being applied, ensuring no data loss on crashes.

### Configuration
```json
{
  "name": "products",
  "dim": 128,
  "wal_enabled": true,
  "wal_sync_on_write": false
}
```

| Option | Default | Description |
|--------|---------|-------------|
| `wal_enabled` | `true` | Enable WAL for crash recovery |
| `wal_sync_on_write` | `false` | Sync to disk after each write (safer but slower) |

### API Endpoints
```bash
# Create checkpoint for a collection
POST /collections/:name/checkpoint

# Get WAL statistics
GET /collections/:name/wal_stats

# Checkpoint all collections
POST /checkpoint_all
```

### Create Checkpoint
Checkpoints save the index state and mark WAL position, allowing old entries to be truncated.

```bash
curl -X POST http://localhost:3000/collections/products/checkpoint
```

Response:
```json
{
  "collection": "products",
  "sequence": 15000,
  "success": true
}
```

### Get WAL Stats
```bash
curl http://localhost:3000/collections/products/wal_stats
```

Response:
```json
{
  "collection": "products",
  "enabled": true,
  "sequence": 15000,
  "last_checkpoint": 10000,
  "entries_since_checkpoint": 5000,
  "file_size": 2048576
}
```

### How It Works
1. **Insert/Delete**: Operation written to WAL first
2. **Apply**: Operation applied to index and storage
3. **Checkpoint**: Index saved, WAL marker written
4. **Recovery**: On startup, replay WAL entries since last checkpoint

### Storage Structure
```
data/products/
├── vectors.bin     # Vector storage
├── index.hnsw.*    # HNSW index files
├── meta.sled/      # Metadata store
├── config.json     # Collection config
└── wal.log         # Write-ahead log
```

### Performance Notes
- WAL adds ~5-10% write overhead
- `sync_on_write: true` ensures durability but reduces throughput by ~50%
- Auto-checkpoint every 10,000 operations by default
- Checkpoint clears WAL, keeping file size bounded

## Batch Upsert

Bulk insert API for ~10x performance improvement over individual inserts.

### API Endpoints
```bash
# Batch upsert to specific collection
POST /collections/:name/upsert_batch

# Batch upsert using default collection
POST /upsert_batch
```

### Request Format
```json
{
  "vectors": [
    {
      "vector": [0.1, 0.2, ...],
      "metadata": {"category": "A", "price": "100"}
    },
    {
      "vector": [0.3, 0.4, ...],
      "metadata": {"category": "B", "price": "200"}
    }
  ]
}
```

### Response Format
```json
{
  "start_id": 0,
  "count": 2,
  "success": true
}
```

### Usage Example
```bash
# Insert 100 vectors at once
curl -X POST http://localhost:3000/collections/products/upsert_batch \
  -H "Content-Type: application/json" \
  -d '{
    "vectors": [
      {"vector": [...], "metadata": {"category": "electronics"}},
      {"vector": [...], "metadata": {"category": "books"}},
      ...
    ]
  }'
```

### Python Example
```python
import requests

vectors = [
    {"vector": [random.random() for _ in range(128)], "metadata": {"idx": str(i)}}
    for i in range(1000)
]

resp = requests.post(
    "http://localhost:3000/collections/products/upsert_batch",
    json={"vectors": vectors}
)
print(resp.json())  # {"start_id": 0, "count": 1000, "success": true}
```

### Performance Optimizations
| Layer | Optimization |
|-------|-------------|
| **VectorStore** | Single flush after batch (vs per-vector) |
| **MetadataStore** | Atomic sled::Batch write |
| **HnswIndexer** | Uses `parallel_insert` for multi-threaded indexing |
| **PayloadIndex** | Batch index updates |

### Benchmark (100 vectors, 128-dim)
| Method | Time | Throughput |
|--------|------|------------|
| Individual inserts | ~1.5s | ~65 vectors/sec |
| Batch upsert | ~0.15s | ~650 vectors/sec |
| **Speedup** | **~10x** | |

## Streaming Search

Server-side streaming search for large result sets (k > 1000). Returns results in batches via SSE (HTTP) or gRPC server streaming.

### Use Cases
- Large k values that would timeout or consume too much memory
- Progressive loading of results for better UX
- Bandwidth optimization with optional vector/metadata inclusion

### HTTP SSE Endpoint
```bash
POST /collections/:name/stream_search
```

### Request Format
```json
{
  "vector": [0.1, 0.2, ...],
  "k": 10000,
  "batch_size": 100,
  "include_vectors": false,
  "include_metadata": true,
  "filter": {"category": "electronics"},
  "range_filters": [{"op": "gt", "field": "price", "value": 100}]
}
```

### Response (SSE Events)
```
event: batch
data: {"results":[{"id":1,"score":0.95,"metadata":{"name":"..."}},...],
       "batch_index":0,"total_batches":100,"results_so_far":100,"total_results":10000,"is_last":false}

event: batch
data: {"results":[...],"batch_index":1,...,"is_last":false}

...

event: complete
data: {"total_results":10000,"batches_sent":100,"success":true}
```

### gRPC Streaming
```protobuf
rpc StreamSearch (StreamSearchRequest) returns (stream StreamSearchResponse);
```

### JavaScript Client Example
```javascript
const eventSource = new EventSource('/collections/products/stream_search', {
  method: 'POST',
  body: JSON.stringify({ vector: [...], k: 10000, batch_size: 100 })
});

eventSource.addEventListener('batch', (e) => {
  const batch = JSON.parse(e.data);
  displayResults(batch.results);
  updateProgress(batch.results_so_far / batch.total_results);
});

eventSource.addEventListener('complete', () => {
  eventSource.close();
});
```

### Options
| Field | Default | Description |
|-------|---------|-------------|
| `batch_size` | 100 | Results per batch |
| `include_vectors` | false | Include vector data in results |
| `include_metadata` | false | Include metadata in results |

## Performance Benchmarks

Tested with 128-dimensional vectors, ~2000 vectors, Apple Silicon.

### Sequential Operations (Single Thread)
| Operation | Throughput | Avg Latency | P99 |
|-----------|------------|-------------|-----|
| GET | 2,359 ops/s | 0.42ms | 1.22ms |
| Search (no filter) | 1,388 ops/s | 0.72ms | 1.55ms |
| Search + Non-indexed Filter | 1,521 ops/s | 0.66ms | 1.16ms |
| Search + filter_ids (500) | 914 ops/s | 1.09ms | 4.76ms |
| Search + filter_id_range | 804 ops/s | 1.24ms | 5.11ms |
| Search + Indexed Filter | 640 ops/s | 1.56ms | 8.38ms |
| Search + Multi Indexed | 440 ops/s | 2.27ms | 8.63ms |
| Upsert | 712 ops/s | 1.41ms | 2.98ms |

### Concurrent Operations (20 Threads)
| Operation | Throughput | Avg Latency | P99 |
|-----------|------------|-------------|-----|
| Search (no filter) | 3,610 ops/s | 5.30ms | 11.77ms |
| Mixed Workload (90R/10W) | 2,975 ops/s | 6.57ms | 16.89ms |
| Search + Indexed Filter | 2,405 ops/s | 8.06ms | 16.67ms |

### Concurrent Scaling
| Threads | Search Throughput | Search + Filter |
|---------|-------------------|-----------------|
| 10 | 2,946 ops/s | 2,207 ops/s |
| 20 | 3,610 ops/s | 2,405 ops/s |

### Benchmark Commands
```bash
# Basic benchmark
python benchmark.py

# Full benchmark with payload index tests
python benchmark_full.py
```

## Hybrid Search (Vector + Text)

Combines vector similarity search with BM25 text search for more accurate retrieval.

### Overview

- **Vector Search**: Uses HNSW index for approximate nearest neighbor search
- **Text Search**: Uses BM25 algorithm for full-text search relevance
- **Fusion Methods**:
  - **Weighted**: `score = α × vector_score + (1-α) × text_score`
  - **RRF (Reciprocal Rank Fusion)**: `score = Σ 1/(k + rank)`

### Configuration

Enable text search by specifying `text_fields` when creating a collection:

```bash
curl -X POST http://localhost:3000/collections \
  -H "Content-Type: application/json" \
  -d '{
    "name": "products",
    "dim": 128,
    "distance": "euclidean",
    "indexed_fields": ["category"],
    "numeric_fields": ["price"],
    "text_fields": ["title", "description"]
  }'
```

### Text-Only Search (BM25)

Search using only text relevance:

```bash
curl -X POST http://localhost:3000/collections/products/text_search \
  -H "Content-Type: application/json" \
  -d '{
    "query": "wireless headphones",
    "k": 10,
    "include_metadata": true
  }'
```

Response:
```json
{
  "results": [
    {"id": 42, "score": 8.5, "metadata": {"title": "Premium Wireless Headphones"}},
    {"id": 15, "score": 7.2, "metadata": {"title": "Budget Wireless Earbuds"}}
  ]
}
```

### Hybrid Search

Combine vector similarity with text relevance:

```bash
curl -X POST http://localhost:3000/collections/products/hybrid_search \
  -H "Content-Type: application/json" \
  -d '{
    "vector": [0.1, 0.2, ...],
    "query": "wireless headphones",
    "k": 10,
    "alpha": 0.5,
    "fusion_method": "weighted",
    "include_scores": true,
    "include_metadata": true
  }'
```

Response:
```json
{
  "results": [
    {
      "id": 42,
      "combined_score": 0.85,
      "vector_score": 0.9,
      "text_score": 0.8,
      "metadata": {"title": "Premium Wireless Headphones"}
    }
  ],
  "total_candidates": 20
}
```

### Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `vector` | float[] | required | Query vector |
| `query` | string | required | Text query for BM25 |
| `k` | int | required | Number of results |
| `alpha` | float | 0.5 | Vector weight (0.0 = text only, 1.0 = vector only) |
| `fusion_method` | string | "weighted" | "weighted" or "rrf" |
| `filter` | object | {} | Metadata filter |
| `range_filters` | array | [] | Numeric range filters |
| `include_scores` | bool | false | Include individual scores |
| `include_metadata` | bool | false | Include metadata in results |

### BM25 Algorithm

The BM25 scoring formula:

```
score(D, Q) = Σ IDF(qi) × (tf(qi, D) × (k1 + 1)) / (tf(qi, D) + k1 × (1 - b + b × |D|/avgdl))
```

Where:
- `IDF(qi)` = inverse document frequency of term i
- `tf(qi, D)` = term frequency in document D
- `k1 = 1.2` = term frequency saturation parameter
- `b = 0.75` = length normalization parameter
- `|D|` = document length
- `avgdl` = average document length

### Use Cases

| Scenario | Recommended Method |
|----------|-------------------|
| Image similarity | Vector only (alpha=1.0) |
| Keyword search | Text only (text_search) |
| Product search | Hybrid (alpha=0.5) |
| Document retrieval | Hybrid with RRF |
| FAQ matching | Hybrid (alpha=0.7) |

### Performance Considerations

- Text indexing adds ~10-20% overhead to insert operations
- BM25 search is O(q × n) where q = query terms, n = matching docs
- Hybrid search runs vector and text search in parallel
- RRF fusion is slightly faster than weighted (no score normalization)

## SIMD Acceleration

Hardware-accelerated distance functions for 4-8x faster vector operations.

### Overview

The SIMD module (`src/simd/mod.rs`) provides:
- **AVX2** (256-bit, 8 floats) - ~4-8x speedup on modern Intel/AMD
- **SSE4.1** (128-bit, 4 floats) - ~2-4x speedup
- **NEON** (ARM) - Native Apple Silicon support
- **Scalar fallback** - Works on any CPU

### Runtime Detection

SIMD level is detected automatically at startup:

```bash
curl http://localhost:3000/simd
```

Response:
```json
{
  "level": "avx2",
  "l2_implementation": "avx2",
  "dot_implementation": "avx2",
  "features": ["avx2", "avx", "fma", "sse4.1", "sse4.2"]
}
```

### Accelerated Operations

| Operation | Function | Speedup |
|-----------|----------|---------|
| L2 (Euclidean) Distance | `simd::l2_squared()` | 4-8x |
| Dot Product | `simd::dot_product()` | 4-8x |
| Cosine Distance | `simd::cosine_distance()` | 4-8x |
| L2 Distance (uint8) | `simd::l2_squared_u8()` | 4-8x |
| Dot Product (uint8) | `simd::dot_product_u8()` | 4-8x |
| Vector Normalization | `simd::normalize()` | 4-8x |

### Usage

The SIMD functions are used automatically:
- **HNSW Search**: Distance calculations use SIMD
- **Quantized Distance**: uint8 comparisons use SIMD
- **Cosine Similarity**: Normalization and dot product use SIMD

### Manual Usage

```rust
use vectordb::simd;

// Auto-detect best implementation
let distance = simd::l2_squared(&vec_a, &vec_b);
let dot = simd::dot_product(&vec_a, &vec_b);
let cosine = simd::cosine_similarity(&vec_a, &vec_b);

// Quantized vectors
let dist_u8 = simd::l2_squared_u8(&quantized_a, &quantized_b);

// Normalize for cosine similarity
let normalized = simd::normalize(&vector);
```

### Architecture Support

| Architecture | SIMD Level | Notes |
|--------------|------------|-------|
| x86_64 (Intel/AMD) | AVX2 + FMA | Most modern CPUs since 2013 |
| x86_64 (older) | SSE4.1 | Fallback for older CPUs |
| aarch64 (Apple Silicon) | NEON | Native ARM SIMD |
| Other | Scalar | Functional but slower |

### Benchmark Impact

Tested with 128-dimensional vectors:

| Operation | Scalar | SIMD (AVX2) | Speedup |
|-----------|--------|-------------|---------|
| L2 Distance | 150ns | 25ns | 6x |
| Dot Product | 120ns | 20ns | 6x |
| L2 (uint8, 128-dim) | 100ns | 15ns | 6.7x |
| 1000 vector search | 15ms | 3ms | 5x |

## Distributed Sharding

Horizontal data partitioning for scaling across multiple collections or nodes.

### Architecture

```
Client Request
     │
     ▼
┌─────────────┐
│ShardRouter  │ ─── Consistent hash ring
└─────────────┘
     │
     ├──────────────┬──────────────┐
     ▼              ▼              ▼
┌─────────┐   ┌─────────┐   ┌─────────┐
│ Shard 0 │   │ Shard 1 │   │ Shard 2 │
│(local)  │   │(local)  │   │(remote) │
└─────────┘   └─────────┘   └─────────┘
```

### Key Components

1. **ConsistentHashRing**: Routes vectors to shards with virtual nodes for even distribution
2. **ShardRouter**: Manages shard configurations and routing decisions
3. **ShardedCollection**: Cross-shard search with parallel result merging
4. **Global ID**: Encodes shard ID + local ID into single u64

### Global ID Encoding

```rust
// 16-bit shard ID + 48-bit local ID
pub fn encode_global_id(shard_id: u32, local_id: u64) -> u64 {
    ((shard_id as u64) << 48) | (local_id & 0x0000_FFFF_FFFF_FFFF)
}

pub fn decode_global_id(global_id: u64) -> (u32, u64) {
    let shard_id = (global_id >> 48) as u32;
    let local_id = global_id & 0x0000_FFFF_FFFF_FFFF;
    (shard_id, local_id)
}
```

### Sharding Strategies

| Strategy | Description | Use Case |
|----------|-------------|----------|
| `HashById` | Hash vector ID to determine shard | Default, even distribution |
| `RangeById` | ID ranges assigned to shards | Sequential access patterns |
| `HashByCollection` | Hash collection name | Multi-tenant isolation |
| `Manual` | Explicit shard assignment | Custom routing logic |

### HTTP API

```bash
# Get sharding status
GET /sharding/config

# Update sharding configuration
POST /sharding/config

# List all shards
GET /sharding/shards

# Get shard info
GET /sharding/shards/:shard_id

# Add a new shard
POST /sharding/shards

# Remove a shard
DELETE /sharding/shards/:shard_id

# Get shard for a vector ID
GET /sharding/route/:vector_id
```

### Example: Initialize Local Sharding

```rust
use vectordb::sharding::{ShardingConfig, ShardRouter};
use vectordb::api::http::init_local_sharding;

// Create 4 local shards for "products" collection
init_local_sharding(4, "products");
// Creates: products_shard_0, products_shard_1, etc.
```

### Example: Multi-Node Configuration

```rust
use vectordb::sharding::{ShardingConfig, ShardConfig, ShardingStrategy};

let config = ShardingConfig {
    strategy: ShardingStrategy::HashById,
    num_shards: 3,
    replication_factor: 1,
    virtual_nodes: 100,  // For balanced distribution
    shards: vec![
        ShardConfig {
            id: 0,
            node: "node-1".to_string(),
            collection: "products_0".to_string(),
            replicas: vec![],
            is_primary: true,
        },
        ShardConfig {
            id: 1,
            node: "node-2".to_string(),
            collection: "products_1".to_string(),
            replicas: vec![],
            is_primary: true,
        },
        ShardConfig {
            id: 2,
            node: "node-3".to_string(),
            collection: "products_2".to_string(),
            replicas: vec![],
            is_primary: true,
        },
    ],
};
```

### Cross-Shard Search

```rust
// ShardedCollection handles parallel search across all shards
let sharded = ShardedCollection::new(
    "products",
    router,
    collection_manager,
);

// Search runs in parallel across all local shards
let results = sharded.search(&query_vector, k, filter)?;
// Results merged and sorted by distance
```

### Consistent Hashing Benefits

- **Minimal Redistribution**: Adding/removing shards moves only ~1/n of data
- **Virtual Nodes**: 100+ vnodes per shard ensures even distribution
- **Deterministic**: Same key always routes to same shard

### Scaling Considerations

| Shards | Vectors per Shard (1B total) | Memory per Shard (128-dim, f32) |
|--------|------------------------------|--------------------------------|
| 4 | 250M | 128 GB |
| 8 | 125M | 64 GB |
| 16 | 62.5M | 32 GB |
| 32 | 31.25M | 16 GB |

## Multi-Vector Search

Search with multiple query vectors simultaneously and fuse results using various methods.

### Use Cases

- **Query Expansion**: Search with multiple representations of a concept
- **Multi-modal Search**: Combine text and image embeddings
- **Ensemble Search**: Use multiple embedding models
- **ColBERT-style**: Late interaction with token-level vectors

### Fusion Methods

| Method | Description | Best For |
|--------|-------------|----------|
| `rrf` | Reciprocal Rank Fusion | Different score scales |
| `sum` | Sum scores from all queries | Comparable scores |
| `max` | Maximum score across queries | OR-style semantics |
| `avg` | Average scores | Balanced aggregation |
| `min` | Minimum score | AND-style (must match all) |

### HTTP API

```bash
POST /collections/:name/multi_search
```

### Request Format

```json
{
  "vectors": [
    [0.1, 0.2, 0.3, ...],
    [0.4, 0.5, 0.6, ...],
    [0.7, 0.8, 0.9, ...]
  ],
  "k": 10,
  "fusion": "rrf",
  "weights": [0.5, 0.3, 0.2],
  "rrf_k": 60,
  "oversample": 2,
  "filter": {"category": "electronics"},
  "include_per_query_scores": true,
  "include_metadata": true
}
```

### Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `vectors` | float[][] | required | Multiple query vectors |
| `k` | int | required | Number of results |
| `fusion` | string | "rrf" | Fusion method (rrf, sum, max, avg, min) |
| `weights` | float[] | null | Per-query weights for weighted fusion |
| `rrf_k` | float | 60.0 | RRF k parameter |
| `oversample` | int | 2 | Fetch k × oversample candidates per query |
| `filter` | object | {} | Metadata filter |
| `include_per_query_scores` | bool | false | Include individual query scores |
| `include_metadata` | bool | false | Include metadata in results |

### Response Format

```json
{
  "results": [
    {
      "id": 42,
      "score": 0.0328,
      "query_matches": 3,
      "per_query_scores": [0.95, 0.87, 0.92],
      "metadata": {"title": "Product A"}
    },
    {
      "id": 15,
      "score": 0.0312,
      "query_matches": 2,
      "per_query_scores": [0.88, null, 0.85],
      "metadata": {"title": "Product B"}
    }
  ],
  "total_candidates": 45,
  "fusion_method": "rrf"
}
```

### Fusion Method Details

#### RRF (Reciprocal Rank Fusion)
```
score = Σ weight_i / (k + rank_i + 1)
```
- Robust to different score scales
- Default k = 60 (higher = less emphasis on top ranks)
- Best for combining results from different embedding models

#### Weighted Sum
```
score = Σ weight_i × score_i
```
- Works well when scores are comparable
- Use weights to prioritize certain query vectors

### Python Example

```python
import requests

# Multi-modal search: text + image embeddings
text_embedding = embed_text("wireless headphones")
image_embedding = embed_image("headphones.jpg")

response = requests.post(
    "http://localhost:3000/collections/products/multi_search",
    json={
        "vectors": [text_embedding, image_embedding],
        "k": 10,
        "fusion": "rrf",
        "weights": [0.6, 0.4],  # Weight text higher
        "include_metadata": True
    }
)

results = response.json()["results"]
for r in results:
    print(f"ID: {r['id']}, Score: {r['score']:.4f}, Matches: {r['query_matches']}")
```

### Rust Usage

```rust
use vectordb::index::multi_vector::{
    MultiVectorQuery, FusionMethod, parallel_multi_search
};

// Create multi-vector query
let query = MultiVectorQuery::new(
    vec![vec1, vec2, vec3],  // Query vectors
    10,                       // k
)
.with_fusion(FusionMethod::RRF)
.with_weights(vec![0.5, 0.3, 0.2])
.with_rrf_k(60.0)
.with_oversample(3);

// Execute parallel search
let results = parallel_multi_search(&indexer, &query, None)?;

for r in results {
    println!("ID: {}, Score: {:.4}, Matches: {}",
             r.id, r.score, r.query_matches);
}
```

### Performance

- Queries executed in parallel using Rayon
- Oversample factor controls candidate pool size
- Fusion is O(n) where n = total unique candidates

| Queries | Oversample | Candidates | Fusion Time |
|---------|------------|------------|-------------|
| 2 | 2 | ~2000 | ~1ms |
| 5 | 2 | ~5000 | ~2ms |
| 10 | 3 | ~15000 | ~5ms |

## Sparse Vectors

Support for sparse vectors enables hybrid dense+sparse retrieval, combining semantic similarity with lexical matching.

### Use Cases

- **Learned Sparse Representations**: SPLADE, DeepImpact, EPIC
- **Lexical Features**: TF-IDF, BM25-style retrieval
- **Keyword Matching**: Exact term matching combined with semantic search
- **Hybrid Retrieval**: Best of both dense and sparse worlds

### Sparse Vector Format

Sparse vectors are stored as sorted (index, value) pairs:

```json
{
  "indices": [0, 15, 42, 100, 512],
  "values": [0.5, 1.2, 0.8, 2.1, 0.3]
}
```

### HTTP API

#### Insert Sparse Vector
```bash
POST /collections/:name/sparse_upsert
```

```json
{
  "indices": [0, 15, 42],
  "values": [0.5, 1.2, 0.8],
  "metadata": {"category": "electronics"}
}
```

#### Batch Insert
```bash
POST /collections/:name/sparse_upsert_batch
```

```json
{
  "vectors": [
    {"indices": [0, 15], "values": [0.5, 1.2], "metadata": {"type": "A"}},
    {"indices": [42, 100], "values": [0.8, 2.1], "metadata": {"type": "B"}}
  ]
}
```

#### Sparse Search
```bash
POST /collections/:name/sparse_search
```

```json
{
  "indices": [15, 42, 100],
  "values": [1.0, 0.5, 0.3],
  "k": 10,
  "filter": {"category": "electronics"},
  "include_metadata": true
}
```

Response:
```json
{
  "results": [
    {"id": 42, "score": 2.5, "metadata": {"category": "electronics"}},
    {"id": 15, "score": 1.8, "metadata": {"category": "electronics"}}
  ]
}
```

### Hybrid Dense + Sparse Search

Combine dense vector similarity with sparse lexical matching:

```bash
POST /collections/:name/hybrid_dense_sparse
```

```json
{
  "dense_vector": [0.1, 0.2, ...],
  "sparse_indices": [15, 42, 100],
  "sparse_values": [1.0, 0.5, 0.3],
  "k": 10,
  "dense_weight": 0.6,
  "sparse_weight": 0.4,
  "fusion": "weighted",
  "include_scores": true,
  "include_metadata": true
}
```

Response:
```json
{
  "results": [
    {
      "id": 42,
      "score": 0.85,
      "dense_score": 0.12,
      "sparse_score": 2.5,
      "metadata": {"title": "Product A"}
    }
  ],
  "fusion_method": "weighted"
}
```

### Fusion Methods

| Method | Description | Best For |
|--------|-------------|----------|
| `weighted` | Linear combination of normalized scores | General purpose |
| `rrf` | Reciprocal Rank Fusion | Different score scales |
| `max` | Maximum score across methods | OR-style semantics |

### Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `dense_weight` | float | 0.5 | Weight for dense score |
| `sparse_weight` | float | 0.5 | Weight for sparse score |
| `fusion` | string | "weighted" | Fusion method |
| `include_scores` | bool | false | Include component scores |

### Rust Usage

```rust
use vectordb::sparse::{SparseVector, SparseIndex, fuse_hybrid_results, HybridSearchConfig};

// Create sparse vector
let sv = SparseVector::from_pairs(vec![(0, 1.0), (15, 2.0), (42, 0.5)]);

// Create sparse index
let index = SparseIndex::new();
index.insert(0, sv.clone())?;

// Search
let query = SparseVector::from_pairs(vec![(15, 1.0), (42, 1.0)]);
let results = index.search(&query, 10);

// Hybrid search
let config = HybridSearchConfig {
    dense_weight: 0.6,
    sparse_weight: 0.4,
    ..Default::default()
};
let hybrid = fuse_hybrid_results(&dense_results, &sparse_results, &config, 10);
```

### Storage Efficiency

Sparse vectors only store non-zero elements:

| Dense Dim | Avg NNZ | Dense Size | Sparse Size | Savings |
|-----------|---------|------------|-------------|---------|
| 30,000 | 100 | 120 KB | 0.8 KB | 99.3% |
| 30,000 | 500 | 120 KB | 4 KB | 96.7% |
| 30,000 | 1000 | 120 KB | 8 KB | 93.3% |

### Performance

| Operation | Complexity | Notes |
|-----------|------------|-------|
| Insert | O(nnz) | nnz = non-zero elements |
| Search | O(q × avg_posting_len) | q = query nnz |
| Hybrid Fusion | O(n) | n = total candidates |

## Vector Versioning

Track vector changes over time with full version history and time-travel queries.

### Features

- **Version History**: Every insert/update creates a new version
- **Time-Travel Queries**: Query vectors at any point in time
- **Rollback**: Revert to any previous version
- **Snapshots**: Get all vectors as they existed at a timestamp
- **Diff**: Compare versions to see changes

### HTTP API

#### Versioned Upsert
```bash
POST /collections/:name/versioned/upsert
```

```json
{
  "vector_id": 42,
  "vector": [0.1, 0.2, ...],
  "metadata": {"category": "electronics"},
  "description": "Updated pricing data"
}
```

Response:
```json
{
  "version_id": 5,
  "vector_id": 42,
  "timestamp": "2024-03-21T10:30:00Z",
  "is_deleted": false,
  "change_description": "Updated pricing data"
}
```

#### Get Latest Version
```bash
GET /collections/:name/versioned/vectors/:id
```

#### Get Version History
```bash
GET /collections/:name/versioned/vectors/:id/history
```

Response:
```json
[
  {"version_id": 1, "timestamp": "2024-03-20T09:00:00Z", ...},
  {"version_id": 3, "timestamp": "2024-03-20T14:00:00Z", ...},
  {"version_id": 5, "timestamp": "2024-03-21T10:30:00Z", ...}
]
```

#### Time-Travel Query
```bash
POST /collections/:name/versioned/vectors/:id/at
```

```json
{
  "timestamp": "2024-03-20T12:00:00Z"
}
```

Returns the version that was active at that timestamp.

#### Rollback to Version
```bash
POST /collections/:name/versioned/vectors/:id/rollback
```

```json
{
  "to_version_id": 3
}
```

Creates a new version with the content from version 3.

#### Snapshot at Timestamp
```bash
POST /collections/:name/versioned/snapshot
```

```json
{
  "timestamp": "2024-03-20T12:00:00Z"
}
```

Returns all active (non-deleted) vectors as they existed at that timestamp.

#### Get Statistics
```bash
GET /collections/:name/versioned/stats
```

Response:
```json
{
  "total_vectors": 1000,
  "active_vectors": 950,
  "deleted_vectors": 50,
  "total_versions": 5000,
  "avg_versions_per_vector": 5.0,
  "next_version_id": 5001
}
```

### Rust Usage

```rust
use vectordb::versioning::{VersionedVectorStore, VersioningConfig, VersionDiff};

// Create versioned store
let config = VersioningConfig {
    max_versions: 100,      // Keep up to 100 versions per vector
    keep_deleted: true,     // Keep deletion markers
    retention_days: Some(30), // Compact versions older than 30 days
};
let store = VersionedVectorStore::with_config(128, config);

// Insert creates version 1
let v1 = store.upsert(0, vec![1.0; 128], HashMap::new())?;

// Update creates version 2
let v2 = store.upsert(0, vec![2.0; 128], HashMap::new())?;

// Get at timestamp
let at_t1 = store.get_at_timestamp(0, t1);

// Rollback to v1
let v3 = store.rollback(0, v1.version_id)?;

// Get diff between versions
let diff = VersionDiff::compute(&v1, &v2);
println!("Vector changed: {}", diff.vector_changed);
println!("Distance: {:?}", diff.vector_distance);

// Snapshot: all vectors at a point in time
let snapshot = store.snapshot_at(timestamp);

// Compact old versions
let result = store.compact();
println!("Removed {} old versions", result.versions_removed);
```

### Configuration Options

| Option | Default | Description |
|--------|---------|-------------|
| `max_versions` | 100 | Max versions per vector (0 = unlimited) |
| `keep_deleted` | true | Keep deletion marker versions |
| `retention_days` | None | Auto-compact versions older than N days |

### Use Cases

| Scenario | Solution |
|----------|----------|
| Audit trail | Version history shows all changes |
| Undo changes | Rollback to previous version |
| Point-in-time recovery | Snapshot at timestamp |
| Debug regressions | Compare version diffs |
| Data lineage | Track change descriptions |

### Storage Overhead

| Versions/Vector | Overhead |
|-----------------|----------|
| 1 | 1x (no overhead) |
| 10 | ~10x storage |
| 100 | ~100x storage |

Use `max_versions` and `retention_days` to control storage growth.

## IVF-PQ (Billion-Scale Search)

Inverted File with Product Quantization enables efficient search on billion-scale datasets.

### Architecture

```
Query Vector
     │
     ▼
┌─────────────────┐
│ Coarse Quantizer│  K-means (nlist clusters)
│   (IVF layer)   │
└────────┬────────┘
         │ Find nprobe closest clusters
         ▼
┌─────────────────┐
│ Inverted Lists  │  Vectors grouped by cluster
│  [c0][c1]...[cn]│
└────────┬────────┘
         │ Search within clusters
         ▼
┌─────────────────┐
│  PQ Quantizer   │  Compressed vectors (M bytes each)
│   (ADC search)  │
└─────────────────┘
```

### How It Works

1. **Training Phase**:
   - Train IVF: Learn nlist cluster centroids via k-means
   - Train PQ: Learn M codebooks on residuals (vector - centroid)

2. **Indexing Phase**:
   - Assign vector to nearest cluster
   - Compute residual (vector - centroid)
   - Encode residual with PQ → M bytes

3. **Search Phase**:
   - Find nprobe closest clusters
   - Compute ADC distance tables
   - Search inverted lists using lookup tables

### Memory Efficiency

| Vectors | Dimension | Raw Size | IVF-PQ (M=8) | Compression |
|---------|-----------|----------|--------------|-------------|
| 1M | 128 | 512 MB | ~8 MB | 64x |
| 10M | 128 | 5.12 GB | ~80 MB | 64x |
| 100M | 128 | 51.2 GB | ~800 MB | 64x |
| 1B | 128 | 512 GB | ~8 GB | 64x |

### Rust Usage

```rust
use vectordb::ivfpq::{IvfPqIndex, IvfPqConfig, ThreadSafeIvfPq};

// Configure IVF-PQ
let config = IvfPqConfig::for_dim(128)
    .with_nlist(1024)    // 1024 clusters
    .with_nprobe(16)     // Search 16 clusters
    .with_m(8);          // 8 PQ subvectors (8 bytes per vector)

// Create index
let index = ThreadSafeIvfPq::new(config)?;

// Train on sample data (typically 10-100x nlist vectors)
let training_data = load_training_vectors();
index.train(&training_data)?;

// Add vectors
let ids = index.add_batch(&vectors)?;

// Search
let results = index.search(&query, 10)?;
for (id, distance) in results {
    println!("ID: {}, Distance: {:.4}", id, distance);
}

// Adjust nprobe for accuracy/speed tradeoff
let results = index.search_with_nprobe(&query, 10, 32)?;
```

### Configuration

| Parameter | Default | Description |
|-----------|---------|-------------|
| `nlist` | 256 | Number of clusters (coarse centroids) |
| `nprobe` | 8 | Clusters to search (accuracy/speed tradeoff) |
| `m` | 8 | PQ subvectors (compression = dim/m) |
| `ksub` | 256 | Centroids per subvector (typically 256) |
| `kmeans_iters` | 20 | Training iterations |

### Tuning Guidelines

#### nlist (Number of Clusters)
- Rule of thumb: `sqrt(n)` to `4*sqrt(n)` where n = dataset size
- 1M vectors → 1000-4000 clusters
- 1B vectors → 30,000-120,000 clusters

#### nprobe (Clusters to Search)
- Higher = better recall, slower search
- Typical range: 1% to 10% of nlist
- Start with nlist/16 and adjust based on recall

#### M (PQ Subvectors)
- Higher M = better accuracy, more memory
- M=8: 8 bytes per vector, ~90% recall
- M=16: 16 bytes per vector, ~95% recall
- M=32: 32 bytes per vector, ~98% recall

### Accuracy vs Speed Tradeoff

| nprobe | Recall@10 | Search Time (1M vectors) |
|--------|-----------|-------------------------|
| 1 | ~50% | 0.1ms |
| 8 | ~85% | 0.5ms |
| 16 | ~92% | 1ms |
| 32 | ~96% | 2ms |
| 64 | ~98% | 4ms |

### Use Cases

| Dataset Size | Index Type | Notes |
|--------------|------------|-------|
| < 100K | HNSW | Full accuracy, low latency |
| 100K - 10M | HNSW or IVF-PQ | Memory constrained → IVF-PQ |
| 10M - 1B | IVF-PQ | Required for memory efficiency |
| > 1B | IVF-PQ + Sharding | Distribute across nodes |

### Comparison with HNSW

| Aspect | HNSW | IVF-PQ |
|--------|------|--------|
| Memory | 4-8 bytes/dim | 0.5-1 bytes/dim |
| Build Time | Fast | Requires training |
| Search Speed | Very fast | Fast |
| Recall@10 | 99%+ | 85-98% |
| Best For | <10M vectors | >10M vectors |

## GPU Acceleration

Hardware-accelerated distance computation for batch operations using Metal (macOS), CUDA (Linux/Windows), or SIMD fallback.

### Architecture

```
src/gpu/mod.rs
├── GpuBackend        # Backend selection (Metal, Cuda, Cpu, Auto)
├── GpuConfig         # Configuration options
├── GpuCapabilities   # Hardware detection
├── GpuAccelerator    # Core accelerator interface
└── ThreadSafeGpu     # Thread-safe wrapper with OnceCell
```

### Backend Detection

```rust
use vectordb::gpu::*;

// Auto-detect best available backend
let config = GpuConfig {
    backend: GpuBackend::Auto,  // Auto-selects Metal → CUDA → CPU
    batch_threshold: 100,       // Use GPU for batches >= 100
    max_batch_size: 10000,
    prefer_gpu_for_large: true,
};

let accelerator = GpuAccelerator::new(config)?;
println!("Backend: {:?}", accelerator.capabilities());
```

### Distance Computation

```rust
// Batch L2 distance (Euclidean)
let query = vec![0.1, 0.2, 0.3, ...];  // 128-dim
let vectors: Vec<Vec<f32>> = load_vectors();

let distances = accelerator.batch_l2_distance(&query, &vectors)?;

// Batch cosine similarity
let similarities = accelerator.batch_cosine_similarity(&query, &vectors)?;

// Batch dot product
let products = accelerator.batch_dot_product(&query, &vectors)?;
```

### k-NN Search

```rust
// Find top-k nearest neighbors
let (indices, distances) = accelerator.knn_search(
    &query,
    &vectors,
    k: 10,
    metric: DistanceMetric::L2
)?;

// Results sorted by distance (ascending for L2, descending for cosine)
for (idx, dist) in indices.iter().zip(distances.iter()) {
    println!("Vector {}: distance {:.4}", idx, dist);
}
```

### Thread-Safe Usage

```rust
use std::sync::Arc;

// Create thread-safe wrapper
let gpu = Arc::new(ThreadSafeGpu::new(config)?);

// Use from multiple threads
let gpu_clone = Arc::clone(&gpu);
std::thread::spawn(move || {
    let distances = gpu_clone.batch_l2_distance(&query, &vectors)?;
});
```

### SIMD Fallback (CPU)

When GPU is unavailable, uses hardware SIMD:

| Platform | SIMD | Speedup vs Scalar |
|----------|------|-------------------|
| x86_64 | AVX2 | 4-8x |
| x86_64 | SSE4.1 | 2-4x |
| ARM | NEON | 2-4x |
| Fallback | Scalar | 1x |

### Performance Characteristics

| Operation | CPU (AVX2) | Metal (M1) | CUDA |
|-----------|------------|------------|------|
| L2 (10K vectors) | ~2ms | ~0.5ms | ~0.3ms |
| Cosine (10K) | ~2.5ms | ~0.6ms | ~0.4ms |
| k-NN (10K, k=10) | ~3ms | ~0.8ms | ~0.5ms |

### Configuration Options

| Option | Type | Default | Description |
|--------|------|---------|-------------|
| `backend` | GpuBackend | Auto | Backend selection |
| `batch_threshold` | usize | 100 | Min batch size for GPU |
| `max_batch_size` | usize | 10000 | Max vectors per batch |
| `prefer_gpu_for_large` | bool | true | Force GPU for large batches |

### When to Use GPU

| Scenario | Recommendation |
|----------|----------------|
| < 100 vectors | CPU (overhead too high) |
| 100 - 1K vectors | GPU if available |
| > 1K vectors | Strongly prefer GPU |
| Real-time search | GPU for consistent latency |
| Batch reranking | GPU for throughput |

### Integration with IVF-PQ

```rust
// GPU-accelerated IVF-PQ search
let ivfpq = IvfPqIndex::new(config);
let gpu = GpuAccelerator::new(gpu_config)?;

// Use GPU for coarse quantizer search
let cluster_distances = gpu.batch_l2_distance(&query, &centroids)?;
let nearest_clusters = top_k_indices(&cluster_distances, nprobe);

// Use GPU for candidate reranking
let candidates = ivfpq.search_clusters(&query, &nearest_clusters)?;
let final_distances = gpu.batch_l2_distance(&query, &candidates)?;
```

## RaBitQ (1-bit Quantization)

Ultra-compact vector quantization achieving 18x compression with theoretical error bounds.

### Architecture

```
src/rabitq/mod.rs
├── RaBitQuantizer     # Core quantizer with rotation matrix
├── QuantizedVector    # 1-bit packed representation
├── RotationMatrix     # Orthogonal random rotation
├── RaBitQIndex        # Index with optional reranking
└── VectorStats        # Norm/mean/variance for ADC
```

### How It Works

1. **Random Rotation**: Apply orthogonal matrix to normalize distribution
2. **Sign Quantization**: Store only the sign bit (1 bit per dimension)
3. **ADC Distance**: Asymmetric computation (query=float32, candidate=1bit)

### Usage

```rust
use vectordb::rabitq::*;

// Create quantizer
let config = RaBitQConfig {
    dim: 128,
    use_orthogonal: true,
    ..Default::default()
};
let quantizer = RaBitQuantizer::new(config);

// Insert vectors
let id = quantizer.insert(&vector);

// Search
let results = quantizer.search(&query, k: 10);
```

### With Reranking

```rust
// Keep originals for reranking (improves recall)
let index = RaBitQIndex::new(config, keep_originals: true, rerank_k: 3);

// Insert and search
index.insert_batch(&vectors);
let results = index.search(&query, 10);  // Reranks top 30 → returns top 10
```

### Performance

| Metric | Value |
|--------|-------|
| Compression | 18.3x |
| Recall@10 | ~79% |
| Memory (1M vectors, 128d) | ~27 MB |

## DiskANN (SSD-based Index)

Billion-scale vector search using SSD storage with Vamana graph.

### Architecture

```
src/diskann/mod.rs
├── DiskANNIndex       # Main index with mmap
├── DiskANNConfig      # Configuration
├── CompressedNode     # In-memory graph node
├── CompressedVector   # RaBitQ or PQ compressed
└── DiskVector         # Full vector on disk
```

### Storage Layout

```
data/diskann/
├── vectors.bin     # Full vectors (mmap)
├── graph.bin       # Compressed graph
└── medoid.bin      # Entry point
```

### Usage

```rust
use vectordb::diskann::*;

let config = DiskANNConfig {
    dim: 128,
    max_degree: 64,      // Graph connectivity (R)
    build_list_size: 100, // Construction ef (L)
    beam_width: 4,        // Search beam width
    use_rabitq: true,     // Use RaBitQ for compression
    ..Default::default()
};

let index = DiskANNIndex::new(config, "data/diskann".into())?;

// Insert vectors (stored on disk)
for vector in vectors {
    index.insert(&vector)?;
}

// Search (loads from SSD as needed)
let results = index.search(&query, 10)?;

// Persist index
index.save()?;
```

### Performance

| Dataset Size | Memory | Disk | Recall@10 |
|--------------|--------|------|-----------|
| 1M | ~50 MB | 500 MB | ~90% |
| 100M | ~5 GB | 50 GB | ~85% |
| 1B | ~50 GB | 500 GB | ~80% |

## HNSW++ (Dual-Branch Architecture)

Enhanced HNSW with dual-branch partitioning and skip bridges.

### Architecture

```
src/hnswpp/mod.rs
├── HnswPPIndex        # Dual-branch HNSW
├── HnswPPConfig       # Configuration
├── Node               # Graph node with branch assignment
├── BranchStats        # Per-branch statistics
└── DistanceMetric     # L2, Cosine, DotProduct
```

### Key Improvements

1. **Dual-Branch Partitioning**: Vectors assigned to 2 branches based on centroid distance
2. **Skip Bridges**: Cross-branch connections for sparse region traversal
3. **Adaptive Layer Selection**: Better entry point selection

### Usage

```rust
use vectordb::hnswpp::*;

let config = HnswPPConfig {
    dim: 128,
    max_connections: 32,
    max_connections_0: 64,
    ef_construction: 200,
    ef_search: 50,
    enable_dual_branch: true,  // Enable dual-branch
    skip_bridge_threshold: 0.3,
    ..Default::default()
};

let index = HnswPPIndex::new(config);

// Insert
index.insert_batch(&vectors)?;

// Search
let results = index.search(&query, 10);

// Check stats
let stats = index.stats();
println!("Branch 0: {}, Branch 1: {}", stats.branch_0_count, stats.branch_1_count);
println!("Skip bridges: {}", stats.skip_bridges);
```

### When to Use

| Data Distribution | Recommendation |
|-------------------|----------------|
| Uniform random | Standard HNSW |
| Clustered | HNSW++ (dual-branch) |
| Multi-modal | HNSW++ |
| Streaming | Standard HNSW |

## NaviX (Adaptive Filtered Search)

Intelligent filtering strategy selection for varying selectivities.

### Architecture

```
src/navix/mod.rs
├── NaviXFilter        # Core filter engine
├── NaviXConfig        # Configuration
├── FilterSelectivity  # Selectivity estimation
├── FilterStrategy     # PreFilter/PostFilter/Hybrid
└── CandidateManager   # Candidate tracking
```

### Filter Strategies

| Strategy | When | Approach |
|----------|------|----------|
| PreFilter | selectivity < 10% | Filter first, then search within |
| PostFilter | selectivity > 50% | Search all, filter after |
| Hybrid | 10-50% | Adaptive expansion with early termination |
| NoFilter | 100% | Skip filtering |

### Usage

```rust
use vectordb::navix::*;

let config = NaviXConfig {
    base_expansion: 2.0,
    max_expansion: 10.0,
    pre_filter_threshold: 0.1,
    adaptive_expansion: true,
    ..Default::default()
};

let filter = NaviXFilter::new(config);

// Estimate selectivity
let selectivity = FilterSelectivity::from_bitmap(&valid_ids, total_vectors);

// Get recommended strategy
let strategy = filter.recommend_strategy(&selectivity);

// Adaptive search
let results = filter.adaptive_search(
    k: 10,
    &valid_ids,
    total_vectors,
    |n| search_function(n)  // Your search function
);
```

### Expansion Formula

```
expansion = base_expansion * (1 / selectivity).sqrt()
clamped to max_expansion
```

| Selectivity | Expansion (k=10) |
|-------------|------------------|
| 50% | 20 |
| 10% | 63 |
| 1% | 100 (max) |

## Performance Benchmarks (100K Vectors)

### Insert Performance

| Method | Throughput |
|--------|------------|
| Single insert | ~700 vec/sec |
| Batch (100) | ~850 vec/sec |
| Batch (1000) | ~1000 vec/sec |

### Search Performance (Single Thread)

| Operation | QPS | Avg Latency | P99 |
|-----------|-----|-------------|-----|
| Search k=10 | 850 | 1.18ms | 3.96ms |
| Search k=100 | 518 | 1.93ms | 4.91ms |
| Search + Filter | 876 | 1.14ms | 5.36ms |

### Concurrent Search

| Threads | QPS | Avg Latency |
|---------|-----|-------------|
| 10 | 1,697 | 5.69ms |
| 20 | 2,293 | 8.57ms |
| 50 | 2,055 | 22.37ms |

### Cache Performance

| Scenario | QPS | Avg Latency | Improvement |
|----------|-----|-------------|-------------|
| Cold (cache miss) | 250 | 4.01ms | - |
| Warm (cache hit) | 1,196 | 0.84ms | **~5x** |

### Filter Optimization

| Filter Type | QPS | Avg Latency | vs No Filter |
|-------------|-----|-------------|--------------|
| No filter | 115 | 8.74ms | - |
| Indexed filter | 637 | 1.57ms | **5.6x** |
| Multi indexed | 915 | 1.09ms | **8x** |

### Index Comparison

| Index | Memory | Recall@10 | Best For |
|-------|--------|-----------|----------|
| HNSW | High | 99%+ | < 10M, high accuracy |
| HNSW++ | High | 95%+ | Clustered data |
| RaBitQ | Very Low | 79% | Memory constrained |
| DiskANN | Low | 85% | > 100M vectors |
| IVF-PQ | Medium | 90% | 10M-1B vectors |

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/yhc007)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/yhc007)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
