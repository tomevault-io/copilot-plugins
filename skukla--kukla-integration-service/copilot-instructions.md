## kukla-integration-service

> API Mesh and GraphQL patterns

# API Mesh Patterns

## Project Uses GraphQL Query Files

GraphQL queries are stored in `.gql` files and inlined at build time:

```
mesh/
├── config.js              # Mesh sources configuration
├── resolvers.js           # Custom GraphQL resolvers
├── queries/               # GraphQL query files (.gql)
│   ├── get-enriched-products.gql
│   ├── categories-batch.gql
│   ├── inventory-batch.gql
│   └── products-list.gql
├── schema/                # JSON Schema response definitions
│   ├── products-response.json
│   ├── category-batch-resp.json
│   └── inventory-batch-resp.json
└── types/                 # GraphQL type definitions
    └── types.graphql
```

## ✅ CORRECT: Query File Pattern

Store GraphQL queries in `.gql` files:
```graphql
# mesh/queries/get-enriched-products.gql
query GetEnrichedProducts($pageSize: Int, $currentPage: Int) {
  mesh_products_enriched(pageSize: $pageSize, currentPage: $currentPage) {
    products {
      sku
      name
      price
      type_id
      inventory {
        quantity
      }
      categories {
        name
      }
    }
    total_count
    performance {
      executionTime
      apiCalls
    }
  }
}
```

## ✅ CORRECT: JSON Schema Pattern

Define REST API responses with JSON Schema:
```json
// mesh/schema/products-response.json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "type": "object",
  "properties": {
    "items": {
      "type": "array",
      "items": {
        "type": "object",
        "properties": {
          "sku": { "type": "string" },
          "name": { "type": "string" },
          "price": { "type": "number" }
        }
      }
    }
  }
}
```

## Generated Files (Build Time)

These files are generated and should be in .gitignore:
- `mesh-client.js` - Contains inlined GraphQL queries
- `mesh.json` - Complete mesh configuration
- `mesh/resolvers.js` - Generated from templates

## JsonSchema Sources Pattern
- Uses JSON Schema files to define REST API responses
- Located in `mesh/schema/` directory
- Consolidates 200+ REST API calls into 1 GraphQL query
- Native mesh caching for performance
- Custom resolvers for data enrichment

## Usage in Actions

```javascript
// actions/get-products-mesh/index.js
const { getProductsFromMesh } = require('../../lib/commerce/mesh-client');

async function main(params) {
  // mesh-client.js contains the inlined query from .gql file
  const result = await getProductsFromMesh(params, config);
  
  return {
    statusCode: 200,
    body: result.products
  };
}
```

## Best Practices
- Store queries in `.gql` files for syntax highlighting and reusability
- Use JSON Schema for REST API response definitions
- Keep generated files (`mesh-client.js`, `mesh.json`) in .gitignore
- Load and inline queries at build time for performance
- Use fragments for reusable query parts
- Leverage mesh caching capabilities

## ❌ AVOID
- Hardcoding GraphQL queries in JavaScript files
- Manual string concatenation for queries
- Checking in generated files
- Query builder abstractions
- Over-fetching data

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/skukla) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
