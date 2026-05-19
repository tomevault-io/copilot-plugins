## polaris-music

> Polaris is a decentralized, blockchain-anchored music registry with graph-based visualization. It provides a canonical, auditable registry of music creators, releases, tracks, and compositions with a focus on detailed relationship tracking and collaborative data submission.

# CLAUDE.md - AI Assistant Guide for Polaris Music Registry

## Project Overview

Polaris is a decentralized, blockchain-anchored music registry with graph-based visualization. It provides a canonical, auditable registry of music creators, releases, tracks, and compositions with a focus on detailed relationship tracking and collaborative data submission.

**Key Innovation**: Uses "Releases" as the primary accounting unit, with Groups (not individual artists) as the main performance entity. Relationships between Persons and Groups are carefully tracked through MEMBER_OF and GUEST_ON relationships.

### Technology Stack

- **Blockchain**: Antelope/EOS (smart contracts in C++)
- **Graph Database**: Neo4j (GQL-compatible Cypher queries)
- **Backend**: Node.js (ES modules) with Express
- **Frontend**: JavaScript InfoVis Toolkit (JIT) for hyperbolic graph visualization
- **Storage**: IPFS + S3 for event data, Redis for hot cache
- **Indexing**: Substreams (Rust-based blockchain data pipeline)
- **Governance**: Fractally DAO integration for Respect-weighted voting

## Repository Structure

```
polaris-music-registry/
├── contracts/              # Antelope C++ smart contracts
│   └── polaris.music.cpp   # Main contract (anchoring, voting, staking)
│
├── backend/
│   ├── src/
│   │   ├── api/           # Express GraphQL + REST API server
│   │   ├── graph/         # Neo4j schema and operations
│   │   ├── storage/       # Event storage (IPFS, S3, Redis)
│   │   └── indexer/       # Event processor (blockchain → graph)
│   └── test/
│       ├── api/           # API endpoint tests
│       ├── graph/         # Graph database tests
│       ├── storage/       # Storage layer tests
│       ├── e2e/           # End-to-end workflow tests
│       └── performance/   # Load and performance tests
│
├── frontend/
│   ├── src/
│   │   └── visualization/ # JIT-based graph visualization
│   └── public/
│
├── substreams/            # Blockchain event indexing
│   ├── src/              # Rust modules
│   └── proto/            # Protobuf definitions
│
├── tools/
│   ├── import/           # Data import from Discogs, etc.
│   └── migration/        # Database migration scripts
│
├── docs/                 # Detailed implementation specs
│   ├── 01-smart-contract.md
│   ├── 02-graph-database-schema.md
│   ├── 03-event-storage.md
│   └── ... (comprehensive documentation)
│
└── README.md            # User-facing documentation
```

## Core Concepts

### Domain Model

**CRITICAL**: Understand the distinction between these entities:

1. **Person**: Individual musician, producer, engineer, songwriter, etc.
   - Can be a MEMBER_OF multiple Groups over time
   - Can be a GUEST_ON tracks where they're not a group member

2. **Group**: Band, orchestra, ensemble, solo project (collection of Persons)
   - Has MEMBER_OF relationships with clear date ranges and roles
   - PERFORMED_ON tracks as a unit
   - Even solo artists are modeled as "groups of one"

3. **Song**: Musical composition (the written work)
   - Has WROTE relationships to Persons (songwriters)
   - Can have multiple Track recordings

4. **Track**: Specific recording/performance of a Song
   - RECORDING_OF a Song
   - PERFORMED_ON by Group(s)
   - Can have GUEST_ON relationships for non-members
   - Can SAMPLE other Tracks

5. **Release**: Album, EP, Single, Live Performance
   - Contains Tracks via IN_RELEASE relationship
   - IN_MASTER for canonical grouping of re-releases
   - RELEASED by Label

### Members vs. Guests - CRITICAL DISTINCTION

**Members** (`MEMBER_OF` relationship):
- Core group members credited on the album
- Perform on multiple tracks as part of the group
- Listed in liner notes as official band members
- Visualized in RGraph around Group nodes

**Guests** (`GUEST_ON` relationship):
- Individual musicians appearing outside group context
- Session musicians, featured artists, engineers
- Perform on specific tracks or contribute to release
- Visualized as non-colored edges to specific tracks/releases

**Rule of Thumb**: If someone performs on >2/3 of tracks and liner notes don't specify, they're likely members. If unclear, default to guest.

### Event-Driven Architecture

All data changes flow through canonical events:

```
User Submission → Event Creation → Off-chain Storage (IPFS+S3)
                                 ↓
                   Blockchain Anchoring (hash only)
                                 ↓
                   Event Processor → Neo4j Graph Update
                                 ↓
                   Community Voting → Finalization → Rewards
```

**Event Types** (defined in smart contract):
- `21` - CREATE_RELEASE_BUNDLE (full release with groups, tracks)
- `30` - ADD_CLAIM (add data to entity)
- `31` - EDIT_CLAIM (modify existing data)
- `40` - VOTE (vote on submission)
- `41` - LIKE (like a node, track path)
- `50` - FINALIZE (distribute rewards)
- `60` - MERGE_NODE (deduplicate entities)

## Development Workflows

### Adding New Features

1. **Read Documentation First**: Check `docs/` for existing specs
2. **Understand Data Flow**: Events → Storage → Blockchain → Processor → Graph
3. **Test at Each Layer**: Unit → Integration → E2E
4. **Update Schema**: If adding graph entities, update constraints in `backend/src/graph/schema.js`
5. **Update Smart Contract**: If adding event types, modify `contracts/polaris.music.cpp`

### Working with the Graph Database

**Key Files**:
- `backend/src/graph/schema.js` - Neo4j operations and schema initialization
- `docs/02-graph-database-schema.md` - Complete schema documentation

**Important Patterns**:
- All node IDs are deterministic hashes or externally provided IDs
- Use `MERGE` for idempotency (replaying events is safe)
- Every data change creates a `Claim` node for audit trail
- Use transactions for multi-step operations
- Provisional IDs follow pattern: `prov:{type}:{hash}`

**Example Query Pattern**:
```cypher
// Always use parameterized queries
MATCH (g:Group {group_id: $groupId})
MATCH (p:Person)-[m:MEMBER_OF]->(g)
WHERE m.to_date IS NULL OR m.to_date > date()
RETURN p.name, m.role
```

### Working with Smart Contracts

**Key File**: `contracts/polaris.music.cpp`

**Development Flow**:
1. Modify contract locally
2. Test on local testnet first
3. Deploy to testnet for integration testing
4. Get community review before mainnet deployment

**Important Tables**:
- `anchors` - Event hashes and metadata
- `votes` - Voting records with Respect weights
- `respect` - Fractally-synced Respect values
- `stakes` - Token stakes on nodes
- `nodeagg` - Aggregated stake data

### Testing Strategy

**Test Commands** (in `backend/`):
```bash
npm test                    # All tests
npm run test:unit           # Fast unit tests
npm run test:integration    # Database integration tests
npm run test:e2e            # Full workflow tests
npm run test:performance    # Load tests
npm run test:coverage       # Generate coverage report
```

**Testing Conventions**:
- Mock blockchain calls in unit tests
- Use separate test database for integration tests
- E2E tests should use realistic data (Beatles example)
- Performance tests target: <100ms API response, <500ms graph queries

### Code Style Conventions

1. **ES Modules**: Use `import/export`, not `require()`
2. **Async/Await**: Prefer over callbacks or raw Promises
3. **Error Handling**: Always wrap Neo4j transactions in try/catch
4. **Naming**:
   - Variables: `camelCase`
   - Functions: `camelCase`
   - Classes: `PascalCase`
   - Constants: `UPPER_SNAKE_CASE`
   - Neo4j node types: `PascalCase` (Person, Group)
   - Neo4j relationships: `UPPER_SNAKE_CASE` (MEMBER_OF, PERFORMED_ON)

5. **Comments**:
   - Use JSDoc for function documentation
   - Explain "why" not "what" in inline comments
   - Document complex Cypher queries inline

6. **Graph Operations**:
   - Always close Neo4j sessions in `finally` blocks
   - Use session.beginTransaction() for multi-step operations
   - Commit only after all operations succeed
   - Rollback on any error

## Key Files Reference

### Critical Implementation Files

| File | Purpose | Key Functions |
|------|---------|---------------|
| `contracts/polaris.music.cpp` | Smart contract | `put()`, `vote()`, `finalize()`, `stake()` |
| `backend/src/graph/schema.js` | Graph operations | `processReleaseBundle()`, `calculateGroupMemberParticipation()` |
| `backend/src/storage/eventStore.js` | Event persistence | Store/retrieve events from IPFS+S3 |
| `backend/src/indexer/eventProcessor.js` | Blockchain→Graph | Process blockchain events into graph |
| `backend/src/api/server.js` | API endpoints | GraphQL + REST server |

### Documentation Files (Canonical Specs)

| File | Contains |
|------|----------|
| `docs/01-smart-contract.md` | Complete contract implementation |
| `docs/02-graph-database-schema.md` | **CANONICAL** schema definition |
| `docs/03-event-storage.md` | Storage layer architecture |
| `docs/04-event-processor.md` | Event processing logic |
| `docs/07-api-server.md` | API endpoint specifications |

**Note**: When documentation conflicts with code, assume docs represent the intended design and code may be incomplete.

## AI Assistant Guidelines

### When Working on This Codebase

1. **Always Read First**: Check `docs/` and `README.md` before implementing
2. **Understand the Domain**: This is music industry data - relationships matter
3. **Preserve Data Integrity**: Use transactions, ensure idempotency
4. **Think Event-Driven**: Every change is an immutable event
5. **Test Thoroughly**: Graph data is complex - test edge cases

### Common Tasks

#### Adding a New Node Type

1. Add constraint in `schema.js` `initializeSchema()`
2. Add processing logic in `processReleaseBundle()` or new event handler
3. Add GraphQL type in API
4. Update documentation in `docs/02-graph-database-schema.md`
5. Add test cases

#### Adding a New Relationship Type

1. Document in `docs/02-graph-database-schema.md` relationship table
2. Add to graph processing logic in `schema.js`
3. Create Cypher query helper if needed
4. Add test cases for relationship creation and traversal
5. Update visualization if relationship should be displayed

#### Adding a New Event Type

1. Define in smart contract `polaris.music.cpp`
2. Add to event processor in `backend/src/indexer/eventProcessor.js`
3. Add storage handling if needed
4. Update emission formula if event earns rewards
5. Add to documentation in `docs/03-event-storage.md`

#### Working with Visualization

- Frontend uses JIT (JavaScript InfoVis Toolkit)
- Groups display RGraph showing member participation
- Person→Group connections use deterministic colors
- Path tracking for "like" feature (planned)
- See `docs/06-frontend-visualization.md` for details

### Important Invariants to Maintain

1. **Event Immutability**: Events are never modified, only new events created
2. **Hash Determinism**: Same input data must produce same hash
3. **Idempotency**: Replaying an event should not duplicate data
4. **Relationship Consistency**:
   - A Person can't be both MEMBER_OF and GUEST_ON for same track
   - Track must RECORDING_OF exactly one Song
   - Release must have at least one Track via IN_RELEASE
5. **Audit Trail**: Every modification creates a Claim node
6. **Provisional IDs**: Temporary IDs until canonical external ID obtained

### Common Pitfalls to Avoid

1. ❌ **Don't** use synchronous Neo4j driver methods
2. ❌ **Don't** forget to close sessions/transactions
3. ❌ **Don't** modify events after creation
4. ❌ **Don't** skip validation on user input
5. ❌ **Don't** hardcode blockchain account names
6. ❌ **Don't** assume Group members are constant over time
7. ❌ **Don't** mix up Song (composition) and Track (recording)
8. ❌ **Don't** use string concatenation for Cypher queries (SQL injection risk)

### Debugging Tips

**Graph Issues**:
```cypher
// Check node counts by type
MATCH (n) RETURN labels(n)[0] as type, count(*) as count

// Find orphaned nodes
MATCH (n) WHERE NOT (n)--() RETURN n LIMIT 10

// Check relationship distribution
MATCH ()-[r]->() RETURN type(r) as rel, count(*) as count
```

**Blockchain Issues**:
```bash
# Check event anchoring
cleos get table polaris polaris anchors --limit 10

# Check vote records
cleos get table polaris polaris votes --limit 10
```

**Event Processing**:
```bash
# Check processor logs
docker-compose logs -f event-processor

# Verify event retrieval
curl http://localhost:3000/api/events/{hash}
```

## Environment Configuration

**Required Environment Variables**:

```bash
# Blockchain
CHAIN_ID=<network chain id>
RPC_URL=<blockchain rpc endpoint>
CONTRACT_ACCOUNT=<your_contract_account>  # polarismusic for testnet, or your account name

# Graph Database
GRAPH_URI=bolt://localhost:7687
GRAPH_USER=neo4j
GRAPH_PASSWORD=<password>

# Storage
IPFS_URL=http://localhost:5001
S3_ENDPOINT=<s3 endpoint>
S3_ACCESS_KEY=<key>
S3_SECRET_KEY=<secret>
S3_BUCKET=polaris-events

# Redis Cache
REDIS_HOST=localhost
REDIS_PORT=6379
REDIS_PASSWORD=<password>
```

## Quick Start for Development

```bash
# Install dependencies
cd backend && npm install

# Start services (requires Docker)
docker-compose up -d

# Initialize database schema
node -e "import('./src/graph/schema.js').then(m => new m.default(config).initializeSchema())"

# Run tests
npm test

# Start API server
npm run dev

# Start event processor
npm run processor
```

## Deployment Considerations

1. **Database Backups**: Neo4j data is critical - automated daily backups
2. **IPFS Pinning**: Ensure events are pinned to avoid garbage collection
3. **Smart Contract**: Immutable once deployed - test exhaustively
4. **Rate Limiting**: API endpoints should have rate limits
5. **Monitoring**: Track graph query performance, event processing lag

## Integration Points

### External Systems

- **Discogs**: Import tool in `tools/import/discogsImporter.js`
- **Fractally**: Oracle updates Respect values weekly
- **IPFS Gateway**: Public gateway fallback if local node down
- **WharfKit**: Frontend blockchain wallet integration

### API Clients

- Frontend visualization consumes GraphQL + REST
- Data import tools use REST endpoints
- Blockchain contract emits events consumed by processor
- Mobile app (planned) will use GraphQL

## Current Status & Roadmap

**Current Phase**: Foundation (In Progress)
- ✅ Graph database schema designed
- ✅ Smart contract specification complete
- 🔄 Event storage implementation
- 🔄 Basic visualization
- ⏳ Event processor implementation
- ⏳ API server implementation

**Next Phase**: Enhancement
- 📋 "Like" function with path tracking
- 📋 Advanced search capabilities
- 📋 Mobile application
- 📋 IPNS for mutable references

**See README.md** for complete roadmap

## Getting Help

- **Documentation**: Check `docs/` directory first
- **README.md**: User-facing documentation and API examples
- **Smart Contract**: Heavily commented in `contracts/polaris.music.cpp`
- **Schema Documentation**: `docs/02-graph-database-schema.md` is canonical
- **Tests**: Look at test files for usage examples

## Final Notes for AI Assistants

This is a complex, multi-layered system with blockchain immutability constraints. When in doubt:

1. **Preserve existing behavior** - don't refactor unless explicitly asked
2. **Test extensively** - graph data bugs are hard to fix
3. **Document changes** - update relevant docs/ files
4. **Ask clarifying questions** - especially about domain modeling decisions
5. **Consider the full stack** - changes often require updates at multiple layers

The project values:
- **Data integrity** over performance
- **Auditability** over convenience
- **Decentralization** over simplicity
- **Precision** over approximation (music credits matter!)

When implementing features, remember this is building a canonical music registry that artists and industry will rely on. Accuracy and reliability are paramount.

---

**Document Version**: 1.0
**Last Updated**: 2025-12-05
**Maintained By**: Project contributors

For questions or clarifications, refer to README.md or contact the development team.

---
> Source: [PolarisMusic/polaris-music](https://github.com/PolarisMusic/polaris-music) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
