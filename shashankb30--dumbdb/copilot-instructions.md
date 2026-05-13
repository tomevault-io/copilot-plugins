## dumbdb

> This is a simple relational database implemented in C. It supports basic CRUD (Create, Read, Update, Delete) operations, as well as table creation and deletion.

# DumbDB Project Overview

This is a simple relational database implemented in C. It supports basic CRUD (Create, Read, Update, Delete) operations, as well as table creation and deletion.

## Building and Running

The project uses CMake to build. To build and run the project, use the following commands:

```bash
cmake -B build
cmake --build build
cd build && ./DumbDB
```

## Testing
After building the project, run the following:
```bash
cd build && ctest
```

## Development Conventions

The project follows a simple and consistent coding style. The code is well-commented, and the public API is documented in the header files. The indexing system has been refactored to support both hash table and B-Tree indexes. The project currently compiles.

### Project Structure

*   `README.md`: Provides a high-level overview of the project.
*   `CMakeLists.txt`: The build script for the project.
*   `include/rdbms.h`: The main header file for the project, which defines the public API.
*   `src/main.c`: An example of how to use the database.

#### `src/core/`
Contains the core database implementation.
*   `db_core.c`, `db_core.h`: Manages the database lifecycle, including creation, opening, and closing.
*   `memory.c`, `memory.h`: Handles memory management for database objects.
*   `schema.c`, `schema.h`: Manages table schemas, including creation, modification, and deletion.
*   `storage.c`, `storage.h`: Manages the physical storage of data on disk.

#### `src/engine/`
Contains the query engine, index, and serialization implementation.
*   `dml.c`, `dml.h`: Implements Data Manipulation Language (DML) operations like `INSERT`, `UPDATE`, and `DELETE`.
*   `index_algo.c`, `index_algo.h`: Implements the underlying algorithms for different index types (e.g., B-Tree, hash).
*   `index.c`, `index.h`: Manages the creation, insertion, and lookup of indexes.
*   `query.c`, `query.h`: Implements the query execution logic.
*   `serialization.c`, `serialization.h`: Implements data serialization and deserialization.
*   `table.c`, `table.h`: Manages table-level operations.

#### `src/common/`
Contains common data structures and utility functions.
*   `btree.c`, `btree.h`: Implements the B-Tree data structure.
*   `common.h`: Contains common macros and definitions.
*   `errors.c`, `errors.h`: Defines and manages database error codes and messages.
*   `ht.c`, `ht.h`: Implements a hash table data structure.
*   `types.h`: Defines common data types used throughout the database.

---
> Source: [Shashankb30/DumbDB](https://github.com/Shashankb30/DumbDB) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-13 -->
