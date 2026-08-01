## aliyun-tablestore-python-sdk

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

The Aliyun TableStore SDK for Python is a client library that provides access to Alibaba Cloud's TableStore (a NoSQL database service). The SDK supports various operations including table management, data operations, search indexes, secondary indexes, transactions, and timeseries data.

## Architecture and Structure

### Core Components
- `tablestore/client.py`: Contains the main OTSClient and AsyncOTSClient classes that implement the API interfaces
- `tablestore/protocol.py`: Handles the protocol layer, including request/response encoding/decoding and signing
- `tablestore/connection.py`: Manages HTTP connections using urllib3 for sync and aiohttp for async operations
- `tablestore/encoder.py` & `tablestore/decoder.py`: Handle serialization/deserialization of requests and responses
- `tablestore/auth.py`: Implements authentication mechanisms (SignV2, SignV4)
- `tablestore/metadata.py`: Defines data structures and metadata classes

### Data Encoding Formats
- Protobuf: Primary serialization format using generated protobuf files in `tablestore/protobuf/`
- PlainBuffer: Alternative binary format implemented in `tablestore/plainbuffer/`
- FlatBuffer: Used for certain data types, with schemas and generators in `tablestore/flatbuffer/`

### Key Features Supported
- Synchronous and asynchronous clients (OTSClient and AsyncOTSClient)
- Basic CRUD operations (PutRow, GetRow, UpdateRow, DeleteRow)
- Batch operations (BatchGetRow, BatchWriteRow)
- Range queries (GetRange)
- Search indexes with complex query capabilities
- Secondary indexes
- Local transactions
- Timeseries data operations
- SQL queries
- Parallel scan operations
- Aggregations and GroupBy operations

### Authentication and Security
- Support for both V2 and V4 AWS-style signatures
- STS token support for temporary credentials
- SSL/TLS support with configurable versions
- Credential providers for flexible authentication

## Development Commands

### Setup and Dependencies
```bash
# Install poetry if not already installed
pipx install poetry

# Install project dependencies
poetry install
```

### Code Generation
```bash
# Generate protobuf files
sh ./protoc.sh

# Generate flatbuffer files
sh ./flatc.sh
```

### Testing
```bash
# Set required environment variables (IMPORTANT: use test instance only)
export OTS_TEST_ACCESS_KEY_ID=<your access key id>
export OTS_TEST_ACCESS_KEY_SECRET=<your access key secret>
export OTS_TEST_ENDPOINT=<tablestore service endpoint>
export OTS_TEST_INSTANCE=<tablestore instance name>
export OTS_TEST_REGION=<tablestore region>

# Run a specific test
poetry run pytest tests/full_text_search_test.py -v -s

# Run all tests
poetry run pytest tests -v -s
```

### Building Distribution
```bash
# Build wheel distribution
poetry build
```

## Important Notes

### Testing Environment
Tests perform cleanup operations that may delete tables in the specified instance. Always use a dedicated test instance for running tests.

### API Coverage
The SDK covers the full TableStore API including:
- Basic table operations
- Data manipulation
- Search capabilities
- Secondary indexes
- Transaction support
- Timeseries data features
- SQL query support

### Error Handling
- Uses custom exception hierarchy (OTSClientError, OTSServiceError)
- Implements retry policies with configurable strategies
- Includes detailed error information with request IDs

---
> Source: [aliyun/aliyun-tablestore-python-sdk](https://github.com/aliyun/aliyun-tablestore-python-sdk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-23 -->
