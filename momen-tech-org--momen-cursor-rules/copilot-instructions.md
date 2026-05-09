## momen-database-gql-api-rules

> Momen.app's database architecture and GraphQL schema generation rules.


# Momen.app Database to GraphQL Schema Mapping

The GraphQL schema is automatically generated directly from the PostgreSQL data model.

> **CRITICAL CATCH-ALL RULE:**
> If you are ever in doubt about the exact structure, available fields, Enum values (like `[Enum:ROUNDING_MODE]`), or specific arguments for an endpoint, **you must use the webfetch/curl tool to introspect the live GraphQL schema** via the provided endpoint URL before making assumptions.
 

## 1. Naming Conventions & Root Operations

Each table generates corresponding root query, mutation, and subscription fields. For a table named `[table]`:

### Queries
*   `[table]`: Fetch lists (supports `where`, `order_by`, `distinct_on`, `limit`, `offset`).
*   `[table]_by_pk`: Fetch single record by primary key (`id`).
*   `[table]_aggregate`: Aggregate queries (`count`, `sum`, `avg`, `min`, `max`).
*   `[table]_group_by`: Group records based on specified fields.
*   `fz_[table]_by_[column]`: Auto-generated spatial proximity search if the table contains a `geo_point` column.
*   **Relay API**: If configured, generates a cursor-based pagination query returning a `Connection_[table]` object.

### Mutations
*   `insert_[table]`, `insert_[table]_one`: Create records. Supports nested inserts and `on_conflict` (requires `constraint` and `update_columns`).
*   `update_[table]`, `update_[table]_by_pk`: Modify records. Uses `_set` (replace) and `_inc` (increment numeric). `where` is required (`!`) for bulk updates. *(Note: Hasura JSONB update operators like `_append`/`_delete_key` are not supported).*
*   `delete_[table]`, `delete_[table]_by_pk`: Remove records. `where` is required (`!`) for bulk deletes.
*   `export_[table]`: Trigger a data export task for the table.

### Subscriptions
*   `[table]`, `[table]_by_pk`, `[table]_aggregate`: Live queries mirroring their standard query counterparts.

## 2. Column Types and Data Mappings

### Primitive Types
*   `text` → `String`, `integer` → `Int`, `bigint`, `bigserial` → `bigint`, `float8` → `Float8`, `decimal` → `Decimal`, `boolean` → `Boolean`, `jsonb` → `jsonb`.
*   **Time/Date**: `timestamptz`, `timetz`, `date`, `interval`.
*   **Geo**: `geo_point` → `geography`.

### Composite (Media) Types
Media columns (`image`, `file`, `video`) are structurally 1:N relations but act as fields. 
*   **Single Media**: Stored as `[column]_id` (e.g., `cover_image_id`) referencing system asset tables (`FZ_Image`, `FZ_File`, `FZ_Video`).
*   **Media Lists**: Types like `image_list`, `video_list`, `file_list` map to GraphQL Arrays and are stored as `[column]_ids` (e.g., `gallery_images_ids`).

### System-Managed Columns
`id`, `created_at`, `updated_at` are read-only and automatically managed. They cannot be used in mutation inputs (`_set`, `_inc`, insert inputs).

## 3. Relationships

Relationships are defined by foreign keys and determine GraphQL nested fields:
*   **1:1 (One-to-One)**: Yields a single nested object (e.g., `meta: post_meta`).
*   **1:N (One-to-Many)**: Yields an array (e.g., `post_tags: [post_tag]`) and an aggregate object (e.g., `post_tags_aggregate`).

## 4. Filtering (`where` clauses)

Filters rely on `[table]_bool_exp`.

### Logical Operators
*   `_and: [bool_exp]`, `_or: [bool_exp]`, `_not: bool_exp`

### Relation Filters
Navigate relationships using the relationship field name directly. The value is a nested `[related_table]_bool_exp` object.
*   **To-One Relationships (1:1, N:1)**: Filters the parent record based on the single related record's fields.
    *(e.g., Find posts where the author's name is "John": `author: { name: { _eq: ... } }`)*
*   **To-Many Relationships (1:N, N:M)**: Uses `EXISTS` semantics. The parent record is returned if **any** related record in the array matches the nested condition.
    *(e.g., Find posts that have at least one tag named "Tech": `post_tags: { tag: { name: { _eq: ... } } }`)*

### Comparison Predicates (Strict Pattern)
Momen uses a strict **Operator-First Pattern**. A predicate must start with the operator. If it's a generic operator, the operand wrapper type must match the *final evaluated type*, not necessarily the column type.

**Structure:**
```json
{
  "_operator": {
    "operand_type": {
      "left_operand": { ... },
      "right_operand": { ... }
    }
  }
}
```

*   **Operators:** 
    *   **Comparison (Generic):** `_eq`, `_neq`, `_gt`, `_lt`, `_gte`, `_lte`
    *   **Array (Generic):** `_in`, `_nin`
    *   **Nullity (Generic, Unary):** `_is_null`, `_is_not_null`
    *   **Text (String Pattern):** `_like`, `_nlike`, `_ilike`, `_nilike`, `_similar`, `_nsimilar`
    *   **JSONB:** `_contains`, `_contained_in`, `_has_key`, `_has_keys_any`, `_has_keys_all`
    *   **Boolean:** `_is_true`, `_is_false`
    *   **Collection:** `_is_empty`, `_is_not_empty`
*   **Operand Definitions (`left_operand` / `right_operand`):**
    *   **Literal**: `{"literal": value}`
    *   **Column**: `{"column": "field_name"}`
    *   **Function**: `{"function_name": { ...args }}`
*   **Operand Types (Determined by Final Value):** `bigint_operand`, `text_operand`, `boolean_operand`, `timestamptz_operand`, etc.

**Example Predicate (Extract Month from Timestamp and check if = 12):**
```json
{
  "_eq": {
    "bigint_operand": {
      "left_operand": {
        "extract_timestamptz": { "time": { "column": "created_at" }, "unit": "MONTH" }
      },
      "right_operand": { "literal": "12" }
    }
  }
}
```

## 5. Aggregations, Window Functions, and Order By

### Aggregation (`[table]_aggregate`)
Returns an aggregate query object containing two main fields:
*   `nodes`: An array of the actual table objects (`[table!]!`) containing the raw data rows that match the query's `where`, `limit`, `offset`, and `order_by` criteria.
*   `aggregate`: An object containing statistical calculations over the matched rows:
    *   `count(columns: [Enum], distinct: Boolean)`: 
        *   If no columns are provided, it counts all rows (`COUNT(*)`).
        *   If `distinct: true` and 1+ columns are provided, it counts unique values or unique combinations of the specified columns.
        *   If `distinct: false` and multiple columns are provided, the system restricts the count to only evaluate the *first* column in the array.
    *   `sum`, `avg`: Only available for **numeric** columns.
    *   `max`, `min`: Available for **comparable** columns (numeric, time, text).

### Window Functions
Advanced analytical operations like `ROW_NUMBER`, `RANK`, `DENSE_RANK`, `NTH_VALUE` are supported in specific formula and window frame inputs.

### Sorting (`order_by`)
Supports sorting by:
1.  Direct columns: `{ title: asc }`
2.  Related 1:1 records: `{ author: { name: desc } }`
3.  Aggregates of N:1 records: `{ post_tags_aggregate: { count: desc } }`
4.  Vector Search: Text columns may support similarity sorting if the `TEXT_COLUMN_VECTOR_SORT` extension is applied.

## 6. Formula Functions (Operands)
Functions are used inside operand wrappers by wrapping the uppercase function name around its arguments (e.g., `{"EXTRACT_TIMESTAMPTZ": { "time": ..., "unit": ... }}`).

### Input Constraints & Semantics
*   **[TYPE]**: Indicates a required column or scalar operand of that exact type.
*   **[TYPE?]**: Indicates an optional operand (usually defaults to `0` or `null`).
*   **[ANY]**: Accepts any column type.
*   **[NUMERIC]**: Accepts `BIGINT`, `INTEGER`, `DECIMAL`, `FLOAT8`, or `BIGSERIAL`.
*   **[COMPARABLE]**: Accepts `NUMERIC` types, `TEXT`, `DATE`, `TIMESTAMPTZ`, `TIMETZ`, or `INTERVAL`.
*   **[ANY[]]**: Accepts an array operand of any type.
*   **[Enum:NAME]**: Requires an explicit Enum value matching the `[NAME]` definition (e.g., `[Enum:DATE_UNIT]` requires `YEAR`, `MONTH`, etc.).
*   **Nested Functions**: Arguments can often be the output of other functions, provided the output type matches the expected input type.

### Manipulation
*   `CONCAT(items: [TEXT[]])`
*   `SUBSTRING(source_text: [TEXT], start_index: [BIGINT], end_index: [BIGINT])`
*   `LEFT(source_text: [TEXT], length: [BIGINT])`
*   `RIGHT(source_text: [TEXT], length: [BIGINT])`
*   `LOWER(source_text: [TEXT])`
*   `UPPER(source_text: [TEXT])`
*   `TRIM(text: [TEXT])`
*   `TRIM_TRAILING_ZERO(source_text: [TEXT])`
*   `REPEAT(text: [TEXT], times: [BIGINT])`
*   `ENCODE_URL(text: [TEXT])`
*   `DECODE_URL(text: [TEXT])`
*   `ARRAY_CONCAT(first_array: [ANY[]], second_array: [ANY[]])`
*   `SLICE(array: [ANY[]], start_index: [BIGINT], length: [BIGINT])`
*   `UNIQUE(array: [ANY[]])`
*   `COALESCE(array: [ANY[]])`

### Search & Replace
*   `REPLACE_OCCURRENCES(source_text: [TEXT], search_text: [TEXT], replace_text: [TEXT], max_replacements: [BIGINT])`
*   `REPLACE_AT_POSITION(source_text: [TEXT], start_index: [BIGINT], length: [BIGINT], replace_text: [TEXT])`
*   `POSITION(source_text: [TEXT], search_text: [TEXT])`
*   `CONTAINS(source_text: [TEXT], search_text: [TEXT])`

### Regex
*   `REGEX_EXTRACT(text: [TEXT], regex: [TEXT])`
*   `REGEX_REPLACE(text: [TEXT], regex: [TEXT], replacement: [TEXT])`
*   `REGEX_EXTRACT_ALL(text: [TEXT], regex: [TEXT])`
*   `REGEX_MATCH(text: [TEXT], regex: [TEXT])`

### Formatting & Utils
*   `TEXT_DECIMAL_FORMAT(number: [DECIMAL], fraction_digits: [BIGINT], rounding_mode: [Enum:ROUNDING_MODE], clear_trailing_zeros: [BOOLEAN])`
*   `NUMBER_FORMAT(number: [DECIMAL], fraction_digits: [BIGINT], format: [Enum:NUMBER_FORMAT])`
*   `STRING_LEN(source_text: [TEXT])`
*   `RANDOM_TEXT(min_length: [BIGINT], max_length: [BIGINT], include_numbers: [BOOLEAN], include_lower_case: [BOOLEAN], include_upper_case: [BOOLEAN])`
*   `UUID()`
*   `JOIN(array: [TEXT[]], separator: [TEXT])`
*   `SPLIT(source_text: [TEXT], delimiter: [TEXT])`
*   `ARRAY_LEN(array: [ANY[]])`

### Arithmetic
*   `ADD(value0: [NUMERIC], value1: [NUMERIC])`
*   `SUBTRACT(minuend: [NUMERIC], subtrahend: [NUMERIC])`
*   `MULTIPLY(value0: [NUMERIC], value1: [NUMERIC])`
*   `DIVIDE(dividend: [DECIMAL], divisor: [DECIMAL])`
*   `MODULO(dividend: [NUMERIC], divisor: [NUMERIC])`
*   `ABS(number: [NUMERIC])`
*   `POW(base: [NUMERIC], exponent: [NUMERIC])`
*   `LOG(base: [DECIMAL], argument: [DECIMAL])`

### Rounding & Formats
*   `ROUND_UP(number: [DECIMAL])`
*   `ROUND_DOWN(number: [DECIMAL])`
*   `DECIMAL_FORMAT(number: [DECIMAL], fraction_digits: [BIGINT], rounding_mode: [Enum:ROUNDING_MODE])`

### Generators
*   `RANDOM_BIGINT(min_length: [BIGINT], max_length: [BIGINT])`
*   `SEQUENCE(start: [BIGINT], end: [BIGINT], step: [BIGINT])`

### Current Time
*   `CURRENT_DATE()`
*   `CURRENT_TIMETZ()`
*   `CURRENT_TIMESTAMPTZ()`

### Constructors
*   `MAKE_DATE(years: [BIGINT], months: [BIGINT], days: [BIGINT])`
*   `MAKE_TIMETZ(hours: [BIGINT?], minutes: [BIGINT?], seconds: [BIGINT?], milliseconds: [BIGINT?])`
*   `MAKE_TIMESTAMPTZ(years: [BIGINT], months: [BIGINT], days: [BIGINT], hours: [BIGINT?], minutes: [BIGINT?], seconds: [BIGINT?], milliseconds: [BIGINT?])`
*   `MAKE_INTERVAL(years: [BIGINT], months: [BIGINT], weeks: [BIGINT], days: [BIGINT], hours: [BIGINT], minutes: [BIGINT], seconds: [BIGINT], milliseconds: [BIGINT])`
*   `FROM_DATE_AND_TIMETZ(date: [DATE], timetz: [TIMETZ])`

### Extraction & Formatting
*   `EXTRACT_DATE(time: [DATE], unit: [Enum:DATE_UNIT])`
*   `EXTRACT_TIMETZ(time: [TIMETZ], unit: [Enum:TIME_UNIT])`
*   `EXTRACT_TIMESTAMPTZ(time: [TIMESTAMPTZ], unit: [Enum:TIMESTAMP_UNIT])`
*   `DATE_FORMAT(time: [DATE], format: [TEXT], language: [Enum:LANGUAGE])`
*   `TIMETZ_FORMAT(time: [TIMETZ], format: [TEXT], language: [Enum:LANGUAGE])`
*   `TIMESTAMPTZ_FORMAT(time: [TIMESTAMPTZ], format: [TEXT], language: [Enum:LANGUAGE])`
*   `DURATION_FORMAT(duration: [DECIMAL], unit: [Enum:DURATION_UNIT], format: [TEXT])`
*   `RELATIVE_DATE(time: [DATE], language: [Enum:LANGUAGE], hide_suffix: [BOOLEAN])`
*   `RELATIVE_TIMESTAMPTZ(time: [TIMESTAMPTZ], language: [Enum:LANGUAGE], hide_suffix: [BOOLEAN])`

### Calculations & Deltas
*   `DELTA_DATE(date: [DATE], increase: [BOOLEAN], years: [BIGINT?], months: [BIGINT?], days: [BIGINT?])`
*   `DELTA_TIMETZ(timetz: [TIMETZ], increase: [BOOLEAN], hours: [BIGINT?], minutes: [BIGINT?], seconds: [BIGINT?], milliseconds: [BIGINT?])`
*   `DELTA_TIMESTAMPTZ(timestamptz: [TIMESTAMPTZ], increase: [BOOLEAN], years: [BIGINT?], months: [BIGINT?], days: [BIGINT?], hours: [BIGINT?], minutes: [BIGINT?], seconds: [BIGINT?], milliseconds: [BIGINT?])`
*   `EXTRACT_DATE_DURATION(start_time: [DATE], end_time: [DATE], unit: [Enum:DATE_UNIT])`
*   `EXTRACT_TIMETZ_DURATION(start_time: [TIMETZ], end_time: [TIMETZ], unit: [Enum:TIME_UNIT])`
*   `EXTRACT_TIMESTAMPTZ_DURATION(start_time: [TIMESTAMPTZ], end_time: [TIMESTAMPTZ], unit: [Enum:TIMESTAMP_UNIT])`

### Conversions
*   `FROM_TIMESTAMPTZ_TO_DATE(timestamptz: [TIMESTAMPTZ])`
*   `FROM_TIMESTAMPTZ_TO_TIMETZ(timestamptz: [TIMESTAMPTZ])`

### Geography
*   `FROM_COORDINATES(latitude: [DECIMAL], longitude: [DECIMAL])`
*   `GEO_DISTANCE(point0: [GEO_POINT], point1: [GEO_POINT], unit: [Enum:GEO_DISTANCE_UNIT])`
*   `GEO_LONGITUDE(geo: [GEO_POINT])`
*   `GEO_LATITUDE(geo: [GEO_POINT])`

### Aggregates
*   `SUM(array: [NUMERIC[]])`
*   `AVG(array: [NUMERIC[]])`
*   `MAX(value0: [COMPARABLE], value1: [COMPARABLE])`
*   `MIN(value0: [COMPARABLE], value1: [COMPARABLE])`
*   `GREATEST(array: [COMPARABLE[]])`
*   `LEAST(array: [COMPARABLE[]])`

### Element Access
*   `ITEM(array: [ANY[]], index: [BIGINT])`
*   `FIRST_ITEM(array: [ANY[]])`
*   `LAST_ITEM(array: [ANY[]])`
*   `RANDOM_ITEM(array: [ANY[]])`
*   `ARRAY_POSITION(array: [ANY[]], item: [ANY])`

### JSONB
*   `JSON_EXTRACT_BY_DOT_NOTATION_JSONPATH(json: [JSONB], path: [TEXT])`

### Casting
*   `CAST_FROM_TEXT(value: [TEXT])`
*   `CAST_COLUMN_TO_TEXT(value: [ANY])`
*   `CAST_ARRAY_TO_TEXT(value: [ANY[]])`
*   `CAST_TO_BIGINT(value: [ANY])`
*   `CAST_TO_DECIMAL(value: [ANY])`

### Vector Search
*   `EMBEDDING_VECTOR_DISTANCE(embedded_text_column: [Enum:EMBEDDING_COLUMN], text: [TEXT], distance_function: [Enum:VECTOR_DISTANCE])`

### System
*   `NULL_VALUE()`

### Explicit Enum Definitions
When an argument requires an `[Enum:NAME]`, you must use one of the following exact string values:
*   **[Enum:DATE_UNIT]**: `YEAR, MONTH, DAY, WEEK`
*   **[Enum:DURATION_UNIT]**: `DAY, HOUR, MINUTE, SECOND, MILLISECOND`
*   **[Enum:EMBEDDING_COLUMN]**: `(Dynamically generated based on columns with TEXT_COLUMN_VECTOR_SORT extension)`
*   **[Enum:GEO_DISTANCE_UNIT]**: `METER, KILOMETER, MILE`
*   **[Enum:LANGUAGE]**: `EN, ZH`
*   **[Enum:NUMBER_FORMAT]**: `THOUSANDS_SEPARATOR, PERCENT`
*   **[Enum:ROUNDING_MODE]**: `HALF_EVEN, HALF_UP, HALF_DOWN, UP, DOWN, CEILING, FLOOR`
*   **[Enum:TIMESTAMP_UNIT]**: `YEAR, MONTH, DAY, HOUR, MINUTE, SECOND, MILLISECOND, WEEK`
*   **[Enum:TIME_UNIT]**: `HOUR, MINUTE, SECOND, MILLISECOND`
*   **[Enum:VECTOR_DISTANCE]**: `EUCLIDEAN, COSINE`

---
> Source: [momen-tech-org/momen-cursor-rules](https://github.com/momen-tech-org/momen-cursor-rules) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-04 -->
