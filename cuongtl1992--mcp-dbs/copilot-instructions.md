## database-implementations

> When implementing or modifying database connectors, follow these guidelines:

# Database Implementation Rules

When implementing or modifying database connectors, follow these guidelines:

## Interface Compliance

All database implementations must fully implement the `Database` interface:

```typescript
export interface Database {
  connect(): Promise<void>;
  disconnect(): Promise<void>;
  query(query: string, params?: any[]): Promise<any>;
  execute(query: string, params?: any[]): Promise<void>;
  getTables(): Promise<string[]>;
  getTableSchema(tableName: string): Promise<TableSchema>;
  getSchema(): Promise<SchemaInfo>;
}
```

## Connection Management

- Always implement proper connection management with explicit connect/disconnect methods
- Release resources in disconnect methods to prevent memory leaks
- Validate connection state before executing queries

```typescript
// ✅ Good
async disconnect(): Promise<void> {
  if (this.pool) {
    await this.pool.end();
    this.pool = null;
  }
}

// ❌ Bad
async disconnect(): Promise<void> {
  await this.pool.end();
}
```

## Error Handling

- Include informative error messages
- Properly propagate database-specific errors
- Use the ensureConnected pattern to check connection state

```typescript
// ✅ Good
private ensureConnected(): void {
  if (!this.pool) {
    throw new Error('Database not connected. Call connect() first.');
  }
}

async query(query: string, params: any[] = []): Promise<any> {
  this.ensureConnected();
  try {
    const result = await this.pool!.query(query, params);
    return result.rows;
  } catch (error) {
    throw new Error(`Query execution failed: ${error.message}`);
  }
}

// ❌ Bad
async query(query: string, params: any[] = []): Promise<any> {
  return this.pool.query(query, params);
}
```

## Configuration

- Use typed configuration objects with clear property names
- Document each configuration property with JSDoc comments
- Use environment variables as a secondary configuration source

```typescript
/**
 * Configuration for PostgreSQL database connection
 */
export interface PostgresConfig {
  /**
   * Database host
   */
  host: string;
  
  /**
   * Database port (default: 5432)
   */
  port?: number;
  
  // Other properties...
}
```

## Query Parameters

- Always use parameterized queries to prevent SQL injection
- Validate parameter types when necessary
- Document parameter requirements

```typescript
// ✅ Good
await db.query('SELECT * FROM users WHERE age > $1', [21]);

// ❌ Bad
await db.query(`SELECT * FROM users WHERE age > ${age}`);
``` 

---
> Source: [cuongtl1992/mcp-dbs](https://github.com/cuongtl1992/mcp-dbs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-20 -->
