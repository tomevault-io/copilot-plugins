## bmr-breegade

> **MUST** use Elasticsearch for full-text search functionality.


# Search Rules

## Search Service Architecture

**MUST** use Elasticsearch for full-text search functionality.

**Search Service Structure:**

```typescript
// src/services/elasticsearch/search.ts
import elastic from "./elastic";
import CustomerSearchIndex from "./indexes/CustomerSearchIndex";

class search {
  static customers() {
    return new CustomerSearchIndex(elastic);
  }
}

export default search;
```

## Elasticsearch Client Configuration

**MUST** configure Elasticsearch client in dedicated file:

```typescript
// src/services/elasticsearch/elastic.ts
import { Client } from "@elastic/elasticsearch";

const elastic = new Client({
  node: process.env.ELASTICSEARCH_URL,
  auth: {
    username: process.env.ELASTICSEARCH_USERNAME,
    password: process.env.ELASTICSEARCH_PASSWORD,
  },
});

export default elastic;
```

## Search Index Pattern

**MUST** create dedicated search index classes for each searchable entity:

```typescript
// src/services/elasticsearch/indexes/CustomerSearchIndex.ts
import { Client } from "@elastic/elasticsearch";
import Customer from "@/domain/entities/Customer";

class CustomerSearchIndex {
  private client: Client;
  private indexName = "customers";

  constructor(client: Client) {
    this.client = client;
  }

  async index(customer: Customer) {
    await this.client.index({
      index: this.indexName,
      id: customer.id,
      document: {
        id: customer.id,
        firstName: customer.firstName,
        lastName: customer.lastName,
        email: customer.email,
        legalStatuts: customer.legalStatuts,
      },
    });
  }

  async search(query: string) {
    const response = await this.client.search({
      index: this.indexName,
      query: {
        multi_match: {
          query,
          fields: ["firstName", "lastName", "email"],
        },
      },
    });

    return response.hits.hits.map((hit) => hit._source);
  }
}

export default CustomerSearchIndex;
```

## Search Index Naming

**MUST** use lowercase, singular names for index names:

- `"customers"` (not `"Customers"` or `"customer"`)
- `"products"` (not `"Products"` or `"product"`)
- `"orders"` (not `"Orders"` or `"order"`)

## Domain Search Interface

**MUST** define search interfaces in domain layer:

```typescript
// src/domain/search/SearchIndex.ts
interface SearchIndex<T> {
  index(entity: T): Promise<void>;
  search(query: string): Promise<T[]>;
  delete(id: string): Promise<void>;
  update(entity: T): Promise<void>;
}

export default SearchIndex;
```

**MUST** implement domain interfaces in service layer:

```typescript
// src/services/elasticsearch/indexes/CustomerSearchIndex.ts
import SearchIndex from "@/domain/search/SearchIndex";
import Customer from "@/domain/entities/Customer";

class CustomerSearchIndex implements SearchIndex<Customer> {
  // Implementation here
}
```

## Search Query Patterns

**MUST** use appropriate Elasticsearch query types:

**Multi-field search:**

```typescript
async search(query: string) {
  const response = await this.client.search({
    index: this.indexName,
    query: {
      multi_match: {
        query,
        fields: ["firstName", "lastName", "email"],
        type: "best_fields",
      },
    },
  });

  return response.hits.hits.map(hit => hit._source);
}
```

**Filtered search:**

```typescript
async searchByStatus(query: string, status: string) {
  const response = await this.client.search({
    index: this.indexName,
    query: {
      bool: {
        must: [
          {
            multi_match: {
              query,
              fields: ["firstName", "lastName", "email"],
            },
          },
        ],
        filter: [
          {
            term: {
              status,
            },
          },
        ],
      },
    },
  });

  return response.hits.hits.map(hit => hit._source);
}
```

## Index Management

**MUST** provide index management methods:

```typescript
class CustomerSearchIndex {
  async createIndex() {
    const exists = await this.client.indices.exists({
      index: this.indexName,
    });

    if (!exists) {
      await this.client.indices.create({
        index: this.indexName,
        settings: {
          number_of_shards: 1,
          number_of_replicas: 0,
        },
        mappings: {
          properties: {
            firstName: { type: "text" },
            lastName: { type: "text" },
            email: { type: "keyword" },
            legalStatuts: { type: "keyword" },
          },
        },
      });
    }
  }

  async deleteIndex() {
    await this.client.indices.delete({
      index: this.indexName,
    });
  }
}
```

## Document Operations

**MUST** provide CRUD operations for search documents:

```typescript
class CustomerSearchIndex {
  async index(customer: Customer) {
    await this.client.index({
      index: this.indexName,
      id: customer.id,
      document: customer,
    });
  }

  async update(customer: Customer) {
    await this.client.update({
      index: this.indexName,
      id: customer.id,
      doc: customer,
    });
  }

  async delete(id: string) {
    await this.client.delete({
      index: this.indexName,
      id,
    });
  }

  async get(id: string) {
    const response = await this.client.get({
      index: this.indexName,
      id,
    });

    return response._source;
  }
}
```

## Error Handling

**MUST** handle Elasticsearch errors appropriately:

```typescript
async search(query: string) {
  try {
    const response = await this.client.search({
      index: this.indexName,
      query: {
        multi_match: {
          query,
          fields: ["firstName", "lastName", "email"],
        },
      },
    });

    return response.hits.hits.map(hit => hit._source);
  } catch (error) {
    if (error.meta?.statusCode === 404) {
      // Index doesn't exist
      return [];
    }
    throw error;
  }
}
```

## Search Synchronization

**MUST** synchronize database changes with search index:

```typescript
// In service layer after database operations
class CustomerService {
  async createCustomer(data: Partial<Customer>): Promise<Customer> {
    const customer = await customerRepository.create(data);

    // Sync with search index
    await search.customers().index(customer);

    return customer;
  }

  async updateCustomer(id: string, data: Partial<Customer>): Promise<Customer> {
    const customer = await customerRepository.update(id, data);

    // Sync with search index
    await search.customers().update(customer);

    return customer;
  }

  async deleteCustomer(id: string): Promise<void> {
    await customerRepository.delete(id);

    // Remove from search index
    await search.customers().delete(id);
  }
}
```

## Environment Variables

**MUST** use environment variables for Elasticsearch configuration:

```env
ELASTICSEARCH_URL=https://your-elasticsearch-cluster.com
ELASTICSEARCH_USERNAME=your-username
ELASTICSEARCH_PASSWORD=your-password
```

## Search Response Format

**MUST** return consistent response format:

```typescript
interface SearchResult<T> {
  results: T[];
  total: number;
  page?: number;
  pageSize?: number;
}

async search(query: string, page = 1, size = 10): Promise<SearchResult<Customer>> {
  const response = await this.client.search({
    index: this.indexName,
    query: { /* query */ },
    from: (page - 1) * size,
    size,
  });

  return {
    results: response.hits.hits.map(hit => hit._source),
    total: response.hits.total.value,
    page,
    pageSize: size,
  };
}
```

## Index Mapping Best Practices

**MUST** define appropriate field mappings:

```typescript
mappings: {
  properties: {
    // Use "text" for full-text search fields
    firstName: { type: "text" },
    lastName: { type: "text" },

    // Use "keyword" for exact match fields
    email: { type: "keyword" },
    status: { type: "keyword" },

    // Use "date" for timestamp fields
    createdAt: { type: "date" },

    // Use "long" for numeric fields
    age: { type: "long" },
  },
}
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/bmarguerite)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/bmarguerite)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
