## snowflake-connector-nodejs

> Standards for writing and modifying test files: TypeScript usage, test naming conventions, and mock server URL rules

### Code Rules

When creating new test files, write them in TypeScript.

When writing test names, don't use "should"
❌ `it('should pass the check')`
✅ `it('passes the check')`

When sending API requests to mock servers, always use snowflake.com domains instead of random sites like test.com to prevent accidental data leakage.
❌ `https://test.com/api/endpoint`
❌ `https://example.com/api/endpoint`
✅ `https://snowflake.com/api/endpoint`

---
> Source: [snowflakedb/snowflake-connector-nodejs](https://github.com/snowflakedb/snowflake-connector-nodejs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-26 -->
